#Brainforge documentation
Brainforge is an Artificial Neural Networking library implemented in **Python**, which only depends on
**NumPy**. The interface is intended to look like Keras' interface, but no compilation is done in the
background.

##Models
A Neural Network can be considered as a stack of Layer instances. The following models are implemented in Brainforge:

###Network
The base of all models, this class is a linear stack of Layer instances.

####Constructor parameters
- *input_shape*: tuple, specifiing the dimensionality of the data. An InputLayer will be automatically instanciated
based on this information.
- *layers*: some iterable, holding Layer instances. Optional, layers can also be added to the network via the **add()**
method.
- *name*: string, specifiing a name for the network, used in the **describe()** and **save()** methods.

####Methods
For building the architecture:
- *add*: expects a Layer instance, which it adds to the top of the layer stack.
- *pop*: deletes the last layer from the layer stack.
- *finalize*: finalizes the model, making it ready to fit the data.
  - *cost*: string or CostFunction instance, specifiing the cost (or loss) function used to evaluate the network's
performance. See section: **Costs**
  - *optimizer*: string or Optimizer instance, specifiing the optimizing algorithm used to update the parameters of the
layers. See section: **Optimizers**
- *encapsulate*: pickles information about the network, needed to reload it.
- *from_capsule*: factory method used to reconstruct a network from a capsule.

For model fitting:
- *fit*: fits the model to the given data
  - *X*: numpy array containing the training Xs (the independent variables)
  - *Y*: numpy array containing the training Ys (the dependent variables) 
  - *batch_size*: the batch size of stochastic training. 1 for on-line (per-lesson) training. Defaults to 20.
  - *epochs*: the number of epochs to train for. Defaults to 30.
  - *monitor*: iterable, containing strings, specifiing what runtime scores to monitor. Currently, only "acc" is.
Defaults to nothingness (aka void or nihil).
  - *validation*: tuple, containing separate X and Y tensors used to validate the network performance. Defualts to an
empty tuple.
  - *verbose*: integer, specifiing the level of verbosity. Defaults to 1.
  - *shuffle*: bool, if set, the Xs and Ys will be reshuffled before every epoch. Defaults to True.
supported for monitoring the classification accuracy.
- *fit_csxdata*: Wraps **fit**, X, Y and validation data are extracted automatically from the dataframe.
  - frame: csxdata dataframe. See https://github.com/csxeba/csxdata.git

For prediction and forward propagation:
- *prediction*: forward propagates a stimulus (X) throught the network, returning raw predictions.
  - *X*: the stimulus
- *regress*: alias for prediction
- *classify*: wraps prediction by adding a final argmax to the computation, thus predicting a class from raw
probabilities.
- *predict_proba*: alias for prediction
- *evaluate*: evaluates the network's performance on the supplied tensors.
  - *X*: the input tensor
  - *Y*: the target tensor
  - *classify*: bool, determining whether a classification accuracy should be used or only cost determination.

Some utilities:
- *shuffle*: reshuffle the weights of the layers, effectively resetting the net
- *describe*: if the verbose parameter is True, print out useful information about the net. Otherwise return it as string
- *get_weights*: extract the paramters from the layers. If unfold is set, the parameters are returned as one big vector.
- *set_weights*: expects an iterable, ws. Sets the layers' weights according to **ws**. **ws** can be a flattened vector
if this method's **fold** argument is set.
- *gradient_check*: performs numerical gradient check on the supplied X and Y tensors.
  - *epsilon*: specifies the difference, used in the numerical gradient determination. Defaults to 1e-5.
- *output*: property, returning the network's last output.
- *weights*: property, wrapping **get_weights(unfold=False)** as getter and **set_weights(fold=False)** as setter.
- *nparams*: property, returning the total number of parameters in the model.

###Autoencoder
Wraps **Network**, automatically builds the decoder part of an autoencoder, given the encoder part. Has the same
interface as **Network**.

##Layers
Neural networking operations are implemented as *Layer* subclasses and an ANN can be thought of a stack of *Layer*
instances.

###Core layers
These layers are the ones used most regularily when working with ANNs.

- **InputLayer**: passes the input tensor forward, unmodified.
  - No parameters required
  - This layer gets instantiated automatically
- **Flatten**: performs flattening on a *dims* dimensional tensor. Returns a matrix of shape
(*batch_size*, prod(*dims*))
  - No paramters required
- **Reshape**: reshapes the input tensor to a given shape. Keeps the leading dimension,
which corresponds to *batch_size*
  - No parameters required
- **DenseLayer**: just your regular densely connected layer.
  - *neurons*: integer, specifiing the output dimensions
  - *activation*: string or ActivationFunction instance, specifiing the activation function. Defaults to Linear.
- **Activation**: applies a (possibly) nonlinear activation function elementwise on the
input tensor.
  - *activation*: string, specifiing the function to be used. Can be one of *sigmoid, tanh, relu* or *softmax*.

*Softmax* is only available with *categorycal crossentropy (xent)*, because the raw derivatives I worked out for this 
function won't pass the numerical gradient test, so it's implemented the simplified form of *(a - y)*.

###Fancy layers
Some fancy feedforward layers from various scientific articles

- **HighwayLayer**: See Srivastava et al., 2015. This layer applies a gating mechanism to its inputs. This consist of
a forget gate and an input gate, determining the amount of information discarded and kept. Finally, it applies an
output gate, selecting the information to be passed on to the next layer. This operation doesn't change the
dimensionality of the inputs.
  - activation: string or ActivationFunction instance, specifiing the activation function to be applied
- **DropOut**: Discards certain neurons in the training phase, thus improving generalization.
See Srivastava et al., 2014

Dropout is currently faulty!

###Recurrent
Recurrent layers working with multidimensional (time-series) data

- **RLayer**: simple recurrence, output is simply fed back in each timestep.
  - *neurons*: integer, specifiing the output (and the inner state's) shape of the layer.
  - *activation*: string or ActivationFunction instance, specifiing the activation function.
  - *return_seq*: bool, determining whether to return every output (or inner state) generated (True) or to only return
the result of the last iteration (False). Defaults to False.
- **LSTM**: Long-Short Term Memory, see Hochreiter et al., 1997
  - *neurons*: integer, specifiing the output (and the inner state's) shape of the layer.
  - *activation*: string or ActivationFunction instance, specifiing the activation function. 
  - *return_seq*: bool, determining whether to return every output (or inner state) generated (True) or to only return
the result of the last iteration (False). Defaults to False.
- **GRU**: Gated Recurrent Unit, see Chung et al., 2014
  - *neurons*: integer, specifiing the output (and the inner state's) shape of the layer.
  - *activation*: string or ActivationFunction instance, specifiing the activation function.
  - *return_seq*: bool, determining whether to return every output (or inner state) generated (True) or to only return
the result of the last iteration (False). Defaults to False.
- **Reservoir**: an untrainable, specially initialized recurrent layer used in Echo State Networks.
See Reservoir computing and Jaeger, 2007 (Scholarpaedia)
  - *neurons*: integer, specifiing the output (and the inner state's) shape of the layer.
  - *activation*: string or ActivationFunction instance, specifiing the activation function.
  - *return_seq*: bool, determining whether to return every output (or inner state) generated (True) or to only return
the result of the last iteration (False). Defaults to False.

GRU is currenty unstable and doesn't pass the gradient check test.

###Tensor
Feedforward layers working with multidimensional data

- **PoolLayer**: untrainable layer performing the max-pooling operation
  - *fdim*: integer, specifiing the filter dimension. This value will also be used as the stride of the pooling operation.
- **ConvLayer**: performs convolution/cross-correlation on a batch of images by learnable kernels of a given shape.
  - *nfilters*: integer, specifiing the number of filters (neurons) to be learned
  - *filterx*: integer, specifiing the X dimension of the filter
  - *filtery*: integer, specifiing the X dimension of the filter
  - *activation*: string or ActivationFunction instance, specifiing the activation function. Defaults to Linear. It is a
good practice to apply activation only after the pooling operation to ease the computational load.
  - *mode*: "valid" means no padding will be applied to the input tensor, so the output will have a shape smaller than
the inputs. "full" means zero padding will be applied and the output tensor will be bigger than the input tensor.
"same" means only so much padding will be applied, that the output tensor's shape will be the same as the input tensor's

##Optimizers
Currently the following optimizers are implemented:

- SGD
- Momentum (also Nesterov)
- Adagrad
- RMSprop
- Adam

##Costs
The following cost functions are supported:

- Mean Squared Error (MSE)
- Categorical crossentropy (Xent)
- Hinge loss (Hinge)

##Evolution
Support is available for evolutionary parameter optimization. This technique can be used to either evolve optimal
hyperparameters for an ANN or to evolve the parameters (weights and biases) themselves.
The feature can be accessed via the **brainforge.evolution** module, which defines the following class:
- **Population**: abstraction of a population of individuals
  - *loci*: integer, specifiing the number of loci on the individuals' genome (the length of the genome).
  - *fitness_function*: a function reference, which will be called on every individual. This function has to return
a tuple/list/array of fitness values.
  - *fitness_weights*: an array, specifiing the weights used when calculating the weighted sum of the fitness values,
thus creating a **grade** value representing the individuals' overall value.
  - *limit*: integer, specifiing the size of the population (the number of individuals). Defaults to 100.
  - *grade_function*: reference to a function, which, if supplied, will be used instead of weighted sum to determine the
grade (overall value) of individuals. This function has to return a scalar given a tuple of fitness values. Defaults to
weighted sum, if not set.
  - *mate_function*: reference to a function, which, if supplied, will be used to produce an offspring, given two
individuals (genomes). Defaults to per-locus random selection, if not set.

Populations an be run for a set amount of epochs by using the **Population.run()** method. An epoch consists of the
following evolutionary operations:
1. *selection*: determine which individuals survive, based on their grades
2. *mating*: generate candidate individuals in place of the ones, deleted in the previous step.
3. *mutation*: mutation is applied on a per-locus basis.
4. *update*: updates the fitnesses and grades where it is needed.

The **run** method accepts the following parameters:
- *epochs*: the number of epochs to run
- *survival_rate*: the rate of individuals kept in the *selection* step. Defaults to 0.5.
- *mutation_rate*: the probability of a mutation happening on a per-locus basis. Defaults to 0.1.
- *force_update_at_every*: integer, specifiing whether it is needed to force-update every individual, not just the
ones, which are not up-to-date (e.g. offsprings and mutants).
- *verbosity*: integer, specifiing the level of verbosity. 1 prints out run dynamics after each epoch. > 1 prints out
<erbosity-1> number of the top individuals, their genomes and fitnesses.


##Examples
###Fit shallow net to XOR
```python
import numpy as np

from brainforge import Network
from brainforge.layers import DenseLayer

def input_stream(m=20):
    Xs = np.array([[0, 0], [0, 1], [1, 0], [1, 1]])
    Ys = np.array([[1, 0], [0, 1], [0, 1], [1, 0]])

    while 1:
        arg = np.random.randint(len(Xs), size=m)
        yield Xs[arg], Ys[arg]

net = Network(input_shape=(2,), layers=[
    DenseLayer(12, activation="sigmoid"),
    DenseLayer(2, activation="sigmoid")
])
net.finalize(cost="xent", optimizer="adam")

datagen = input_stream(1000)
valid_stream = input_stream(100)

for epoch, (X, Y) in enumerate(datagen, start=1):
    print("Epoch", epoch+1)
    net.epoch(X, Y, batch_size=20, monitor=["acc"],
              validation=next(valid_stream), verbose=1)
    if epoch == 30:
        break
```

For more complicated tasks, the use of the library csxdata is suggested.

###Fit LeNet-like ConvNet to images
```python
from csxdata import CData

from brainforge import Network
from brainforge.layers import (DenseLayer, DropOut, Activation,
                               PoolLayer, ConvLayer)

dataroot = "path/to/pickled/gzipped/tuple/of/X/Y/ndarrays.pkl.gz"
images = CData(dataroot, indeps=0, headers=None)

inshape, outshape = images.neurons_required

model = Network(inshape, layers=(
    ConvLayer(nfilters=10, filterx=3, filtery=3),
    PoolLayer(fdim=2),
    DropOut(0.5),
    Activation("relu"),
    ConvLayer(nfilters=10, filterx=5, filtery=5),
    PoolLayer(fdim=3),
    Activation("relu"),
    DenseLayer(120, activation="tanh"),
    DropOut(0.5),
    DenseLayer(outshape, activation="softmax")
))
model.finalize(cost="xent", optimizer="rmsprop")
X, Y = images.table("learning")
valid = images.table("testing")
model.fit(X, Y, batch_size=20, epochs=30, validation=valid,
          monitor=["acc"])
```

###Fit LSTM to text data
```python
from csxdata import Sequence

from brainforge import Network
from brainforge.layers import DenseLayer, LSTM

datapath = "path/to/text/file.txt"
# Chop up the data into three-character 'ngrams'
# (a character-level RNN would use n_gram=1) 
# Sequence data is ordered. In this case, the
# timestep parameter specifies how many ngrams
# are in X before Y is set. Consider the following case:
# [Mar][y h][ad ][a l][itt][le ][lam]
# [     THESE NGRAMS ALL GO TO X    ]
# The next ngram, [b ,] is the Y corresponding to the
# above X. Thus X is 3-dimensional, conventionally:
# (timestep, batch_size, data_dimensionality),
# Where data_dimensionality either comes from either
# the one-hot representation of every individual ngram,
# or some kind of embedding into k-dimensional continous
# space.
seq = Sequence(datapath, n_gram=3, timestep=7)
inshape, outshape = seq.neurons_required
model = Network(inshape, layers=(
    LSTM(120, activation="relu", return_seq=True),
    LSTM(60, activation="relu", return_seq=False),
    DenseLayer(120, activation="tanh"),
    DenseLayer(outshape, activation="softmax")
))
model.finalize(cost="xent", optimizer="rmsprop")
X, Y = seq.table("learning", shuff=True)
valid = seq.table("testing", shuff=True)
model.fit(X, Y, batch_size=32, epochs=100,
          monitor=["acc"], validation=valid)
```
###Evolve network hyperparameters
```python
import time

from brainforge import Network
from brainforge.layers import DenseLayer
from brainforge.evolution import Population, to_phenotype

from csxdata import CData

dataroot = "path/to/data.csv"
frame = CData(dataroot, headers=1, indeps=1, feature="FeatureName")

inshape, outshape = frame.neurons_required

# Genome will be the number of hidden neurons at two network DenseLayers.
ranges = ((10, 100), (10, 60))
# We determine 2 fitness values: the network's classification accuracy and
# the time required to run the net. These two values will both be minimized
# and the accuracy will be considered with a 20x higher weight. 
fweights = (20, 1)

def phenotype_to_ann(phenotype):
    net = Network(inshape, layers=[
        DenseLayer(int(phenotype[0]), activation="tanh"),
        DenseLayer(int(phenotype[1]), activation="tanh"),
        DenseLayer(outshape, activation="softmax")
    ])
    net.finalize(cost="xent", optimizer="adagrad")
    return net
    
# Define the fitness function
def fitness(genotype):
    start = time.time()
    net = phenotype_to_ann(to_phenotype(genotype, ranges))
    net.fit_csxdata(frame, batch_size=20, epochs=30, verbose=0)
    score = net.evaluate(*frame.table("testing", m=10), classify=True)[-1]
    error_rate = 1. - score
    time_req = time.time() - start
    return error_rate, time_req

# Build a population of 12 individuals
pop = Population(loci=2, limit=12,
                 fitness_function=fitness,
                 fitness_weights=fweights)
# The population is optimized for 12 rounds with the hyperparameters below.
# at every 3 rounds, we force a complete-reupdate of fitnesses, since the
# neural networks contain some random noise due to initialization, random
# Xs, etc.
pop.run(epochs=12,
        survival_rate=0.7,
        mutation_rate=0.05,
        force_update_at_every=3)
# Select the best candidate and convert it to a network
best = phenotype_to_ann(to_phenotype(pop.best))
```

###Evolve network weights and biases
```python
import numpy as np

from matplotlib import pyplot as plt

from brainforge import Network
from brainforge.layers import DenseLayer
from brainforge.evolution import Population

from csxdata import CData
from csxdata.utilities.vectorops import upscale


# 2 fitness values are being used, the classification error rate
# and the L2 norm of the weights.
def fitness(geno):
    pheno = upscale(geno, -10., 10.)
    net.set_weights(pheno)
    cost, acc = net.evaluate(*frame.table("learning", m=35000))
    fness = 1. - acc, np.linalg.norm(pheno)
    return fness

# We define our own grading function. Weighted sum would be unstable,
# because the L2 norm is a way bigger number than the error rate [0. - 1.]
# so instead of summing, we calculate the product of the fitnesses.
def grade_fn(*fitnesses):
    return np.prod(fitnesses)

frame = CData(source="path/to/data.csv", indeps=1)

# We only need a single net, because the Network.set_weights(fold=True)
# can be used to set the network weights to the evolved parameters.
net = Network(frame.neurons_required[0], layers=[
        DenseLayer(30, activation="tanh"),
        DenseLayer(frame.neurons_required[1], activation="sigmoid")
    ])
net.finalize(cost="mse", optimizer="adam")

pop = Population(
    limit=30,
    loci=net.get_weights().size,
    fitness_function=fitness,
    fitness_weights=(1, 1),
    grade_function=grade_fn
)

# At every evolutionary epoch, the population mean and minimum
# grades are recorded, so the run dynamics can be plotted after
# the run.

means, bests = pop.run(epochs=100,
                       survival_rate=0.0,
                       mutation_rate=0.3)
# Plot the run dynamics
Xs = np.arange(1, len(means)+1)
fig, axarr = plt.subplots(2, sharex=True)
axarr[0].plot(Xs, totals)
axarr[1].plot(Xs, means, color="blue")
axarr[1].plot(Xs, bests, color="red")
axarr[0].set_title("Total grade")
axarr[1].set_title("Mean (blue) and best (red) grades")
plt.tight_layout()
plt.show()
```