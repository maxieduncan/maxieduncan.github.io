---
layout: post
category : books
title: "Hands-On Machine Learning with Scikit-Learn and TensorFlow: Part 2"
tags : [book, review, notes, ml, tensorflow]
---
{% include JB/setup %}

It took me a couple of months to get back to Part 2 of [Hands-On Machine Learning with Scikit-Learn and TensorFlow](http://shop.oreilly.com/product/0636920052289.do), Neural Networks and Deep Learning by Aurélien Géron.

It's a useful read, though I would have benefitted more from taking advantage of the hand on exercises and getting a bit of practical experience building out some simple networks. The book makes a good reference and the exercises are something I'd like to get back to. For now it's done a good job of clarifying for me the use cases of the different types of neural networks out there and of what TensorFlow can do.

The impression I came away with is that TensorFlow (like most other ML frameworks) offers a lot of packages out of the box that you can plug together to have a functional neural network. It was eye opening seeing how many different options and optimisations there are, and getting an insight into when you or may not want to use them. The Clustering section (that I mostly skimmed through) gave me the impression that while you have a lot of options to help you scale neural networks, it's not a simple process and is very much dependent on the type of network you are putting together.

Overall I'm impressed with how easy it is put to a neural network together using TensorFlow. Conversely, scaling out and clustering a neural network looks to be quite complex, as does optimising a network which in almost all literature I've read remains something of a black art. At some point it seems that this is something ML will itself solve, selecting the best neural network, optimisations and parameters for the data being trained and already you can find studies and examples of progress being made in this area.

Below are my notes from book, again these are mostly of use only to me and you're betting off buying the book.

---

## Tensorflow Notes
Computations aren't performed immediately but rather are added to the graph for when it's run.

Computation graphs allow different operations to be performed in parallel, can automatically compute gradients for you, allows the same model to easily be run across different hardware and provides a visual representation. Conversely it makes the learning curve steeper and step by step debugging harder.

### Managing Graphs
There is one graph per session.

When evaluating, previous values aren't maintained and will be evaluated each time. To do this efficiently you can ask tensor flow to evaluate multiple values in a single  graph run rather than one at a time.

All node values are dropped between graph runs except for variable values which are maintained by the session. Variable stay around from when initialised until the session ends.

In a distributed clusters, variables live in the container and will be shared across sessions and live as long as the container.

Variables: operation that holds a value and needs to be initialised. You can change the value (using the tf.assign() function) and it's stateful in that it keeps the same value across multiple runs. Typically used to hold model parameters.

Placeholders: just hold information about the type and shape of tensor they're a placeholder for. Typically used to feed training or test data and pass values to do things like change the value of a variable by supplying model weights.

### Feeding Data to the Training Algorithm
Placeholder nodes allow you to supply values at runtime, can optionally enforce the shape.

### Saving and Restoring Models
Once the model is trained you want to save the parameters to disk for future use and comparison.
May want to contemplate checkpoints during intensive training in case of hardware crashes.

## Training Deep Neural Nets
### Vanishing /Exploding Gradients Problem
When using back propagation and calculating gradients, the gradients often get smaller and smaller as we get closer to the lower layers meaning that the weights at these levels don't get changed much. It's also possible for the gradients to increase having the opposite effect where the weights at the lower levels get changed drastically (mostly in Recurrent Neural Networks).

The Xavier or He initialisation strategy can alleviate this and speed up training by helping maintain the signal flow (difference in gradients between inputs and outputs) in both directions (forwards when making predictions, backwards when back propagating gradients).

#### Non-saturating Activation Functions
Sigmoid activation functions match what we've observed in nature but often behave poorly in a DNN. Other activation functions often work better, especially ReLU which doesn't saturate for positive values.

ReLU can cause neurons to be entirely deactivating only ever outputting 0, especially when a large learning rate it used. Variants such as LeakyReLU can alleviate this by ensuring that the nodes never quite die and can become active again.

ELU can provide even better results but isn't as performant during computation but is faster at converging during training. Tensorflow provides `tf.nn.elu`

ELU > LeakyReLU (and variants) > ReLU > tanh > logistic.

#### Batch Normalisation
Adds an operation to zero-centre and normalise the inputs. This is done by calculating the mean and standard deviation of the mini batch. This isn't possible during testing as there is no mini-batch, so the mean and standard deviation of the entire training set is used.

Benefits:
* diminishes vanishing gradients problem
* networks are less sensitive to weight initialisation
* can use larger learning rater, speeding up learning
* reduces the need for other regularisation techniques

Negatives:
* adds complexity to the model
* slower computation means slower predictions at runtime

In Tensorflow the `batch_norm()` does everything for you while `batch_normalization()` requires you to calculate the mean and standard deviation yourself.

#### Gradient Clipping
Essentially limit the gradient values to some threshold. Batch Normalisation is generally preferred over Gradient Clipping.

### Reusing Pre-trained Layers
For many problems you can make use of an existing DNN already trained on a similar problem by reusing the lower levels (those closer to the inputs); this is known as /transfer learning/.

#### Freezing the lower layers
As the lower levels have already been trained to identify the more generic features it often makes sense to freeze them to lower computation costs when training the new more specific layers. The easiest way to do this is only pass the variables for the new layers to the optimiser, essentially freezing the lower level parameters in place.

To take this further you can cache the results of the frozen layer for a set of inputs then provide then as the inputs to the levels that you are training.

#### Unsupervised Pre-training
If you have a lot of unlabelled training data you can try the layers one by one using an unsupervised feature detector algorithm like /Restricted Boltzmann Machines/

Can be useful when you have a complex problem and no similar model to reuse and little labeled training data but plenty of unlabelled.

#### Summary
Four ways to speed up training:
* good initialisation strategy for the connection weights
* good activation function
* Batch Normalisation
* Reusing parts of a pre-trained network

## Faster Optimisers
There are a number of optimisers that can be faster than Gradient Descent:
* *Momentum optimisation*, modification of Gradient Descent that adds a hyper-parameter for momentum to speed getting to the minimum. If the momentum hyper-parameter is too close to 1 it can cause oscillation taking longer to converge
* *Nesterov Accelerated Gradient*, modification of Momentum optimisation that tweaks the gradient that is measured
* *AdaGrad*, good for simple quadratic problems, stops too soon in neural networks
* *RMSProp*, improvement over AdaGrad and better than Momentum optimisation and Nesterov Accelerated Gradient.
* *Adam optimisation*, currently considered the best. Adaptive Moment Estimation combines the ideas of AdaGrad and Momentum optimisation. Requires less tuning of the learning rate hyper-parameter and often the default of 0.001 can be used.

### Learning Rate Scheduling
There are different /learning schedules/ available that manipulate the learning rate being used to reduce the training required.
* *Pre-determined piecewise constant learning rate*, after a certain number of epochs reduces the learning rate
* *Performance scheduling* after N steps, reduces the learning rate by a factor when the error stops dropping
* *Exponential scheduling* the learning rate is reduced by a factor 10 based on the number of iterations
* *Power scheduling* similar to exponential scheduling but the learning rate drops more slowly

AdaGrad, RMSProp and Adam optimisation all automatically reduce the learning rate during training.

## Avoiding Overfitting though Regularisation
### Early stopping
Interrupt the training when its performance on the validation set starts dropping.

Typically you'll get performance by combining it with other regularisation techniques.

### l1 and l2 Regularisation
Constrain a neural networks weights (but typically not it's biases).

Many Tensorflow functions contain `*_regularizer` params that allow you to specify using the `l1 _regularizer`, `l2 _regularizer` or `l1 _l2_regularizer` functions. The losses calculated by these functions then need to be explicitly added to the overall loss.

### Dropout
A percentage of neurons will be left out of a training run. After training neurons are no longer dropped but need to be reduced by the /keep probability/ (1 - drop rate). Tensorflow provides the `tensorflow.contrib.layers.dropout()` function to enable this.

If the model is overfitting you can increase the dropout rate, conversely you can decrease it if underfitting.

Dropout does tend to slow down convergence but the improvement it offers is typically considered worth it.

### Max-Norm Regularisation
Constrains the weights of the incoming connections on each neutron. The max-norm hyper-parameter that controls this can be reduced to increase the amount of regularisation and reduce overfitting. It also helps alleviate vanishing/exploding gradient problems if Batch Normalisation isn't being used. Not provided ootb by Tensorflow but easy to implement.

### Data Augmentation
Generating new data from existing artificially increasing the size of the data set. Adding whit noise doesn't help but transformations such as rotating, shifting and resizing can. This can often be on the fly to reduce the need to for additional storage space.

## Recommended DNN Configuration
Initialisation: *He Initialisation*
Activation function: *ELU*
Normalisation: *Batch Normalistion*
Regularisation: *Dropout*
Optimiser: *Adam*
Learning rate schedule: *None*

Ideally use parts of a similar neural network if one's available.

## Clustered Tensorflow
I have to admit to skimming through this chapter as I'm not at a point where it's of much use to me. It's good to be aware at a high level of what's available but with no plans to run Tensorflow in more than a trivial setting currently it's something I'll revisit when I do need it.

There are numerous configuration options possible but also a number of limitations in how the workload can be spread depending on the model being trained. The core choices it seems to come down to is whether you use synchronous (waiting for all gradients to be calculated) or asynchronous updates (using a gradient as soon as it's been calculated, problematic) and choosing between in graph replication (one graph containing every neural network) or between-graph replication (a graph for every neural network).

## Convolutional Neural Networks
### Convolutional Layer
Partially connected layers.  Good for image processing and visual tasks.

Uses receptive fields that are repetitively applied to smaller subsets of the data set to identify features.

During training, filters that are most useful for the task are discovered and combined into complex patterns to identify specific features. A convolutional layer simultaneously applies numerous filters to its inputs allowing it to find features anywhere in its inputs. Have to define the size, stride and padding type when applying the filter.

Once a CNN has to learned to identify a feature in one location it can also be used to find it in other locations, unlike a DNN which is limited to the location on which it was trained.

Requires a lot of memory as the reverse pass of back propagation requires all the values computed during the forward pass.

### Pooling Layer
Aim is to reduce the input to reduce computational load, memory usage and the number of parameters. This is a destructive layer reducing the amount of data.

Again have only partially connected layers.

Common examples are max pools (only highest value goes forwards) or mean pools (only mean value goes forwards).

### CNN Architectures
Typically a stack of Convolutional layers, then a Pooling layer, then repeat as necessary. So the input gets smaller as you go deeper but more features are mapped. At the end the layers are fully connected.

Well known examples are:
* *LeNet-5*, written digit identification.
* *AlexNet*, similar to LeNet-5 but applied to image identification. Used dropout, augmentation and a competitive normalisation step.
* *GoogLeNet*, use of inception modules to reduce the number of parameters required allowing the network to be much deeper.
* *ResNet*, 152 layer CNN. Uses skip connections where the signal feeding into layer is also added to the output of a layer further up the stack.

## Recurrent Neural Networks
Analyse time series data to make predict future outcomes. Work on sequences of data such as time or sentences making them good for Natural Language Processing.

This can make them interesting creatively, e.g. selecting the next note to use in a musical sequence, or the next word in a sentence.

RNNs pass outputs back into them selves as inputs using two sets of weights, one for the inputs and the second for the outputs of the previous step.

### Memory Cells
Because of the inputs from the previous steps RNNs essentially keep track of previous data.

### Input and Output Sequences
RNNs can take inputs and produce outputs that can be used to make predictions as to what the next value in the sequence will be. E.g. predicting the next word in a sentence.

Alternatively you can just feed in a sequence out inputs and ignore the outputs except for the final one. E.g. predict a rating for the content.

You can also provide a single input and a a sequence of outputs. e.g. caption for an image.

The last option is two step process that takes a sequence of inputs to produce a sequence of outputs, known as Encoder-Decoder. e.g. translation.

### Training RNNs
Process the input sequence then use back propagation, known as Back Propagation Through Time.

### Deep RNNs
Stack multiple layers of cells.

With long sequences, the vanishing/exploding gradient descents can be experienced but can also be alleviated by the previously mentioned solutions: Batch Normalisation, good parameter initialisation, selection of activation function (such as ReLU), gradient clipping and, faster optimisers.

There can be a bigger problem with long sequences and slow training time, the solution is to unroll the RNN only over a limited number of time steps during training which is known as /truncated back propagation through time/. Truncating the input data like this can have unwanted impact on training however, causing long term patterns to be missed.

More generally data at the start of a sequence can often lose its impact over time as it's influence gets diluted over time. A number of cell types have been created to help work around this:

### LTSM Cells
Long Short Term Memory converge faster and detect long term trends in data. It splits the state into two vectors, one is essentially for short term and the other long term. It is made up out of three gates:
* *forget gate*, determines the parts of the long term memory to be erased
* *input gate*, controls which parts of the inputs should be added to the long term state
* *output gate*, selects the parts of the long term state that should be output at this step

#### Peephole Connections
A LSTM cell lets the gate controllers look only at the inputs and the previous short term state, peephole connections give access to the previous long term state as well.

### GRU Cell
Gated Recurrent Unit is a simplified variant of the LSTM cell that performs just as well. Both short and long term state vectors are merged into a single vector. A single gate controller manages both the forget and input gate; if it outputs a 1 then the input gate is open and the forget gate closed, if 0 the opposite. i.e. when a memory is stored, the location where it will be is erased first. There is no output gate, the full vector is output at every step though there is a new gate that controls the parts of the previous state shown to the main layer.

LSTM and GRU cells are the main reason for recent the success of RNNs.

### Natural Language Processing
For Embeddings are used to group words with similar meaning/features. Pre-trained word embeddings are available though you'll likely benefit from refining with training against your own data. This allows the data that is processed to be greatly reduced.

An encoder-decoder network is used. The source is passed in backwards into the encoder so that it will be what gets decoded first. The word embeddings are what actually get passed into the encoder and decoder.

## Autoencoders
Neural Networks that learn efficient representations of input data (called /codings/) without any supervision. This is helpful for dimensionality reduction. Most usefully they act as powerful feature detectors and can be used for unsupervised pre-training of DNNs. They can also be used to generate similar training data based on these features.

So they are useful for:
* dimensionality reduction
* feature extraction
* unsupervised pretraining
* generative models

### Efficient Data Representations
Autoencoders at inputs and converts them into efficient representation.  Made up of two parts: an /encoder (recognition network)/ that converts the inputs into the internal representation and a /decoder (generative network)/ that converts the internal representation into outputs. So the number of outputs will be the same as the number of inputs. The process will drop data that isn't considered important and only keep the the features that are.

Stacked Autoencoders have multiple hidden layers. Adding too many layers will remove the benefits, you can imagine getting to a point where you just have an internal one to one mapping.

You can tie the weights of an encoder to the decoder you make the decoder weights equal to the transpose of the encoder. This reduces the parameters of the model by half leading to faster convergence with less data and reducing the risk of overfitting.

### Denoising Autoencoders
Adding noise to training data to train the autoencoder to learn how to remove it. Can be pure Gaussian noise added to the inputs or randomly switched off inputs (like in Dropout).

### Sparse Autoencoders
Adding a term to the cost function to reduce the number of active neurons in the coding layer. Forces the autoencoder to represent each input as a small number of activations which means that the autoencoder typically ends up representing important features. Aim is good feature extraction.

This is done by penalising neurons whose activation exceeds a particular threshold. This can be done by adding the squared error to the function but a better approach is to use the Kullback-Leibler divergence which has stronger gradients than MSE.

Sum up the sparsity loss for each neurone in the layer add to the result of the cost function.

### Variational Autoencoders
Probabilistic Autoencoders whose outputs are partly determined by chance. The are also Generative Autoencoders.

This makes them similar to RMBs but they are easier to train and sampling is faster.

## Reinforcement Learning
A software /agent/ makes /observations/ and takes /action/ in an /environment/ and in return is /rewarded/ (or /punished/).

### Policy Search
The policy is the software used by the agent to determine its actions.
* *stochastic policy*, using randomness to make choices
* *genetic algorithms*, trying a selection of policies then only retaining a selection that perform well
* *policy gradients*, evaluating the gradients of the rewards then tweaking the parameters

### Neural Network Policies
Rather than just selecting the action that has the best probability of succeeding, you're better off to add some randomness so that the network can explore some of the actions. You need a balance between exploiting actions that are now to have a positive impact and exploring new ones.

### The Credit Assignment Problem
Rewards can be few and far between each step so how do you determine which steps were positive and which weren't?

A common method is to evaluate an action on the sum of all the rewards that come after it, typically applying a discount rate at each step. Bad runs can limit the impact of this on good steps so many runs are required and the scores need to be normalised to ensure that good actions don't lost.

It helps to add rewards where possible for intermediate steps.

### Policy Gradients
PG algorithms optimise the parameters of a policy by following the gradients towards better rewards.
1. Compute the gradients at each step but don't apply them yet
2. Compute each actions score
3. If positive, the action was good and you want to apply the gradients computed in step one; if negative, the action was bad so you want to apply the opposite gradients to make it less likely in the future.
4. Compute the mean of all the resulting gradient vectors and use to perform a Gradient Descent step.

### Markov Design Process
Loops and predicting future states, beyond my ability to properly understand at the moment.

### Temporal Difference Learning and Q-Learning
Temporal Difference Learning is tweaked Value Iteration that takes into account the fact that the agent initially only knows possible states and actions i.e. only limited knowledge of the MDP. It uses an /exploration policy/ to explore the MDP and as it discovers more the TD Learning algorithm updates the estimates of the state values based on the transitions and rewards that are observed.

It's similar to Stochastic Gradient Descent, particularly as it handles one sample at a time.Like SGD can only properly converge if the learning rate is gradually reduced.

Similarly to TD Learning, the Q-Learning algorithm is an adaptation of Q-Value Iteration where the transition probabilities and rewards are initially unknown. For each state-action pair it keeps track of a running average of the rewards the agent gets when leaving the state following an action, plus the rewards it expects to get later. Given enough iterations it will converge on optimal Q-Values. This is known as /off-policy/ algorithm as it learns the optimal policy by just watching an agent act randomly.

### Exploration Policies
Q-Learning only works if it can explore the MDP thoroughly. A purely random policy will get but may take a very long time. A better option is the /Ɛ-policy/ which spends more time on known parts of the graph that are successful while still exploring parts that haven't been visited yet.

Another approach to random chance is to encourage the algorithm to explore actions that it hasn't before which can be done by adding a bonus to the Q-Value estimates.

### Approximate Q-Learning
Q-Learning doesn't scale well to large MDPs with states and actions. The solution is to find a function that approximate Q-Values using a manageable set of parameters, this is known as Approximate Q-Learning. A DNN used to estimate Q-Values is known as a /deep Q-network/ (DQN), using a DQN for Approximate Q-Learning is called Deep Q-Learning.
