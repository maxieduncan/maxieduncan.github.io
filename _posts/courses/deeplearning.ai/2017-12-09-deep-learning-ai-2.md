---
layout: post
category : courses
title: "Deeplearning.ai Notes: Improving Deep Neural Networks"
tags : [course, notes, ml]
---
{% include JB/setup %}

My notes from: https://www.coursera.org/learn/deep-neural-network

## Notation
`X{t}` = batch `t`
`x(i)` = training example `i`
`z(l)` = layer `l`

## Glossary
epoch - pass through a single batch

## Improving Deep Neural Networks
### Practical aspects of Deep Learning
Intuitions ofter don't transfer. Selecting hyperparameters is an iterative process.

*Data*
Split into:
* training set
* dev set (hold out cross validation set)
* test set (for unbiased validation of your model)

For smaller data sets (e.g. 10, 000) split using a ratio like 60/20/20, for larger (e.g. millions) only a small dev/test set is needed such as 98/1/1.

Look out for mismatched training and test data distribution e.g. sourcing training data from the internet but dev and test data from users. Make sure dev and test  come from the data source as training data.

Can be OK to not have a test set if you don't need an unbiased measure of performance.

*Bias/Variance*
High bias = underfitting
High variance = overfitting

Hard to visualise with high number of dimensions so other methods are needed. Comparing the training set error rate against the dev set error rate can help determine this.
* low training error, high dev error = high variance
* high training error, high dev error = high bias
* high training error, even worse dev error = high variance, high bias (over fits only a part of the data)
* low trainman error, low dev error = low variance, low bias

High *bias* (training data performance): *try bigger network*, train longer, even try a new NN architecture, reduce lambda
High *variance* (dev set performance): *try more data*, regularisation, even try a new NN architecture, increase lambda

*Logistic Regression Regularisation*
L2 regularisation: `+ λ/2m (wᵀw)`
L1 regularisation: `+ λ/2m ||w||`, w will be sparse
Typically only regularise w, not b.

`λ` is the regularisation parameter, another hyperparameter to tune.

At the matrix level it's called the "Frobenius norm".

If lambda is large, then reduces W making Z take on a smaller range of values.

*Dropout Regularisation*
Drops a random percentage of nodes from each layer.

Inverted Dropout:
`d[l] = np.random.rand(a[l].shape[0], a[l].shape[1]) < keep_prob`, keep_prob = 0.8 for 80%
`a[l] = np.multiply(a[l], d[l])`
`a[l] = a[l] / keep_prob`, ensures that the expected values remain the same
This will zero (1 - keep_prob) activation nodes.
Can vary keep_prob by layer. Typically layers with more connections you'll want to apply dropout more.

When making predictions at test time you don't use drop out, only during training.

Dropout reduces the dependency on any one feature.

Only need to use dropout if overfitting.

*Other Regularisation Techniques*
Data augmentation: modifying existing data to create new examples.
Early Stopping: when training, compare output with dev set and when comparative performance starts to drift stop.

*Normailsation*
Normalise your training sets.
Subtract the mean:
`u = 1/m sum(x)`
`x = x - u`

Normalist variance:
`σ² = 1/m sum(x*2)`, element-wise multiplication
`x = x / σ²`

Normalise training, dev and test using the same u and σ² values.

Without normalisation, some features will have more of an impact and make optimisation harder. Normalise input features so that they are on a similar scale.

*Vanishing/Exploding Gradients*
Tendency of weights to increase/decrease exponentially, impacting optimisation.

This can be mitigated by careful choice of the initialisation of the weights.
Can make the variance of W to be like 1/n, known as weight decay.
`W[l] = np.random.randn(shape) * np.sqrt(1/n[l-1])`, recommended weight reduction for tanh, Xavier initialisation
`W[l] = np.random.randn(shape) * np.sqrt(2/n[l-1])`, recommended weight reduction for ReLU

*Gradient Checking*
Used to verify the back prop implementation is correct.
Don't use in training, only to debug.
Grad check doesn't work with drop out. Disable dropout to check.
If using regularisation, then include that term.

Take all your weights and bias's, and reshape into a single big vector θ. Do the same with all ∂W and db into dθ.
```
for each i:
	dθapprox[i] = (J(θ1, ..., θi + ε, ...) - J(θ1, ..., θi - ε, ...)) / 2ε
```
This should be approximately equal to `dθ[i] = 2J/2θi`
Check if `dθapprox` is almost equivalent to `dθ`

To do this check:
`||dθapprox - dθ||2 / (||dθapprox||2 + ||dθ||2)`

Use `ε = 10^-7`, if check is `10^-7`, great, `10^-5`, OK, `10^-3`, problem. i.e. looking for a very small value.

If grad check fails, look at the components to try and identify the bug.

### Optimisation Algorithms

*Batch Gradient Descent*
Process the entire training set in a single batch.
Takes a long time for large data sets.
Has memory impacts as the entire data set is loaded.

*Stochastic Gradient Descent*
Mini batch size is 1, so m batches, one for each training example.
Doesn't ever converge on the minimum, will oscillate around it.
Loose the advantages of vectorisation.

*Mini-batch Gradient Descent*
Break your training set up into smaller batches.
Gives you the advantages of both vectorisation and smaller batches.

If you have a small training set (< 2000) just use Batch Gradient Descent.
Typical mini batch sizes: 64, 128, 256, 512.
Make sure mini batch firs in CPU/GPU memory.

A pass through a batch is known as an *epoch*.

*Exponentially weighted averages*
Weights it using previous values. It's an exponentially decaying function, i.e. older values have less and less impact.β
`vt = βvt-1 + (1-β)θt`, where β is the weight, θ is the value. Larger values flatten out.

More efficient than calculating an average normally.

*Bias Correction*
As `v` starts at 0 as there's no previous data, at the start there's a bias to making values lower than they should be.
You can avoid this early bias by dividing `vt` by `1-β^t`.

*Gradient Descent with Momentum*
Typically faster than normal Gradient Descent, uses an exponentially weighted average of the gradient to speed up descent. So as the slope becomes smaller the step size increases to speed up descent.
`v[∂W] = β*v[∂W]+ (1-β)∂W` `v[db] = β*v[db]+ (1-β)db`
`W = W - α*v[∂W]`, `b = b - α*v[db]`

Typical value for `β` is 0.9.
Typically Bias Correction isn't performed, as it isn't significant after 10 iterations of gradient descent.

*RMSprop*
Root Mean Squared prop. Another method that can help speed up Gradient Descent by reducing oscillation.
`s[∂W] = β*s[∂W]+ (1-β)∂W^2` `s[db] = β*s[db]+ (1-β)db^2`
`W = W - α*∂W/sqrt(s[∂W])`, `b = b - α*db/sqrt(s[db])`

RMSprop and Momentum can be used together, meaning that there will be two β hyper parameters, β1 and β2.

*Adam optimisation*
Adaptive Moment Estimation
Works well across a number of areas. Combines Momentum and RMSprop.
`v[∂W] = β1*v[∂W]+ (1-β1)∂W` `v[db] = β1*v[db]+ (1-β1)db`, Momentum
`s[∂W] = β2*s[∂W]+ (1-β2)∂W^2` `s[db] = β2*s[db]+ (1-β2)db^2`, RMSprop
Adam usually adds bias correction:
`v[∂W] = v[∂W] /(1-β1t)` `v[db] = v[∂db] /(1-β1t)`
`s[∂W] = s[∂W] /(1-β2t)` `s[db] = s[db] /(1-β2t)`
Finally perform the update:
`W = W - α*v[∂W]/sqrt(s[∂W]) + ε`, `b = b - α*v[db]/sqrt(s[db]) + ε`

Hyperparameters:
`α` : needs to be tuned
`β1`: 0.9
`β2`: 0.999
`ε`: 10^-8

Some advantages of Adam include:
* Relatively low memory requirements (though higher than gradient descent and gradient descent with momentum)
* Usually works well even with little tuning of hyperparameters (except  α)

*Learning Rate Decay*
Reducing the learning rate over time. Can help stop oscillation when you get nearer the minimum.

`α = (1 / (1 + decay_rate * epoch_num)) * α[0]`, decay_rate is a hyper param, epoch_num is the iteration

There are plenty of other formulas that can be used to decrease the learning rate over epochs.

*Local Optima*
Typically not actually an issue, especially when there are a lot of features.
Plateaus however (more typical) can make learning slow.

### Hyperparameter tuning, Batch Normalization and Programming Frameworks

*Hyperparameter tuning*
In order of importance
* `α` is the most important
* `β` usually 0.9, medium
* Number of hidden units, medium
* Mini batch size, medium
* Number of layers, low
* Learning rate decay, low

Random selection is typically better than using a grid search to select hyper parameters to try. When you find a good area can focus on trying other values in that area.

Scale matters when selecting hyper parameters to test e.g. using a log scale. `r = -4 * np.random.rand()`, `α = 10^r` get numbers from 10^-4 to 10^0, i.e. 0.0001 to 0.1.
For exponentially weighted averages such as `β` 0.9 to 0.9999, `r = -4 * np.random.rand()`, `1-β = 10^r`, `β=1-10^r`.

Babysitting one model, tweaking the hyper parameters as the model trains. Typical when you have limited resources.
Training many models in parallel with different hyper parameters, compare against each other.

*Batch Normalisation*
Normalising the inputs for every layer.
`u = 1/m sum(Z)`, mean
`σ[l] = 1/m sum((Z -u)^2)`, variance
`Znorm = (Z-u) / (sqrt(σ[l]+ε))`, epsilon is added to avoid division by 0
`Z2 = γ[l]Znorm + β[l]`, γ and β are parameters used to scale
If `γ == sqrt(σ[l]+ε)` `β == u`, then Z == Z2

So parameters are: W, b, γ and β (note β is not the same as the hyper-parameter used in Momentum). Can update the γ and β parameters as you do W and b.
Batch Norm will remove the effect of b, so you can eliminate the b parameter.

Tensorflow offers: `tf.nn.batch_normalization`.

Batch Norm keeps the mean and variance the same, even if the values flowing into a layer are changing. It limits the amount of changes in the early layers flowing into the later layers.

Batch Norm also has a regularisation affect. The mean and variance that are computed on the mini batch, as well as the values calculated for z, all adds some noise to the hidden layers activations (similarly to dropout, though to a lesser extent). The larger the mini batch size, the less noise that is produced i.e. less regularisation. So it has a regularisation affect but shouldn't be used for that purpose as it's minimal.

To use at test time, need values for γ and β rather than trying to generate from the test set. To do this track during training and use the γ and β that were learned during and use an exponentially weighted average.

*Multi-class Classification*
Final layer outputs multiple outputs, one for each class.

Softmax activation function: `t = e^(Z), a = t / sum(t)`.
This gives you a probability for each output in the final layer that the input is of the class that output represents.

Softmax regression generalise logistic regression to C classes. If C=2 you have logistic regression.

Calculating loss:
`L(yhat,y) = -sum(ylog(yhat))`

For Back Prop:
`dZ[L] = Yhat - Y`
