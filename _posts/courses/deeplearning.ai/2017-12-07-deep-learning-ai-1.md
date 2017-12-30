---
layout: post
category : course
title: "Deeplearning.ai Notes: Neutral Networks & Deep Learning"
tags : [course, notes, ml]
---
{% include JB/setup %}

My notes from: https://www.coursera.org/learn/neural-networks-deep-learning

## Notation
x = inputs
y = output
ŷ = predicted output
a = predicted output
m = number of training examples
n = number of features
X = input matrix (n by m), so rows are features, columns are training examples
Y = output matrix (1 by m)
w = weights parameter (n sized vector)
b = bias parameter (real number)
L = Loss function
J = Cost function
α = learning rate, controls step size
∂ = partial derivative (two or more variables, can use d for one)
dvar = intermediate derivative values in a computation graph
σ = Sigmoid activation function
L = number of layers in a model
n[l] = number of units in layer l
a[l] = activations in layer l

## Numpy
`np.sum(axis=0)`, vertical sum
`np.sum(axis=1)`, horizontal sum
`np.dot(A, B)`, multiply A with B, *matrix multiplication*: (n,m) * (m,p) = (n,p)
`np.multiply(A, B)` and `A*B`, performs *element wise* multiplication, (n,m) * (n,m) = (n,m)n
`np.exp(x)`, e^x
`np.linalg.norm(x,axis=1,keepdims=True)`, normalise

Tips:
* always explicitly shape matrices; especially avoid rank 1 arrays (i.e. don't use (5) or (5,), use (5,1))
* you can add assert statements to check the shape: `assert(a.shape == (5,1))`

## Neural Networks and Deep Learning
ReLU - Rectified Linear Unit, linear function that starts at 0

Deeply connected: every input is connected to every node

Neural Networks are for supervised learning, where you know the inputs and the expected outputs, i.e. have labelled data.

Structured data: e.g. database, features are well defined
Unstructured dataL e.g. audio, images, text

Sigmoid versus ReLU:
Using Sigmoid as the activation function, the learning rate becomes very slow at the outer regions where the gradient is small. This has been alleviated by the use of ReLU which has a constant gradient of 1. This has drastically improved the performance of gradient descent.

Rises in Deep Learning  due to:
* Data (amount)
* Computation (faster)
* Algorithms (improvements)

### Neural Networks Basics
Logistic Regression, used for binary classification. Generates a probability (between 0 and 1) for an input x , that y will equal 1.

Use the sigmoid function to make sure that values are between 0 and 1.
`σ(z) = 1 / (1 + e^-z)`
If z is large, σ(z) will be close to 1; if z is a large negative number, σ(z) will be close to 0.

*Cost Function*
The aim is for predictions of x to be as close to to y as possible. The difference between the predicted value and the actual value is the cost and the aim is to minimise it.
The Cost Function is the average of the loss function of the entire training set.

Loss Function
Calculates the error for a specific training example x.
Could use the square error for the cost function but this makes gradient descent non optimal, making it harder to optimise the parameters.

Instead use a logarithmic loss function for each training example:
 `L(ŷ,y)= -(y log(ŷ) + (1-y) log(1-ŷ))`.
If y = 1, want ŷ large as `-log(ŷ)`
If y = 0, want ŷ small as `-log(1-ŷ)`

Cost Function
For the parameters of the algorithm, we want to minimise the loss function outputs for the entire training set:
`J(w,b) = 1/m (sum(L(ŷ,y)))`

*Gradient Descent*
Minimise the Cost function to find the global minimum. Expects a convex function, i.e. doesn't have multiple local minima.

```
Repeat {
	w = w - α (∂J(w,b) / ∂w)
	b = b - α (∂J(w,b) / ∂b)
}
```

The derivate is the slope of the function.

*Computation Graph*
Forward propagation allows you to calculate J, while propagating backwards allows you to calculate the derivative of all the inputs.

*Vectorising Logistic Regression*
`Z = wᵀ*X + b = np.dot(w.T, X) +b`
`A = σ(Z)` - vector of predictions
`∂Z = A - Y`
`-1/m * np.sum(Y*np.log(A) + (1-Y)*np.log(1-A))`, cost

`∂w = 1/m * X∂Zᵀ`, n x 1 vector
`∂b = 1/m * sum(∂z) = 1/m * np.sum(∂Z)`

`w = w - α*∂w`
`b = b - α*∂b`

*Python broadcasting*
When adding, subtracting, multiplying or dividing a vector or row vector to a matrix, python will expand the vector to duplicating rows/columns to meet the dimensions.
`(m,n)` +-/* `(1,n) -> (m,n)`
`(m,n)` +-/* `(m,1) -> (m,n)`
Look out for unexpected results caused by broadcasting being used because variables are not in the correct order.

### Shallow Neural Networks

*Activation Functions*
Use tanh `a = (e^z - e^-z)/(e^z + e^-z)` instead of sigmoid `a = 1 / (1 + e^-z)` as the activation function as it's better as centering with outputs between 1 and -1 rather than 1 and 0. The one exception is the output for binary classification where sigmoid can be used for the output layer (as it's 0 or 1).

With both of these when z is either very large or very slow the gradient changes becoming very small slowing down gradient descent. This is where the ReLU (Rectified Linear Unit) function comes in: `a = max(0, z)`. This is the most commonly used function.

Leaky ReLU `a = max(0.01z, z)` has a slight negative slope instead of using 0, i.e. when z is negative then the slope is negative rather than 0.

*Sigmoid activation function*
`g(z) = 1 / (1 + e^-z)`, sigmoid activation function
`g'(z) = a (1 - a)` - derivation of sigmoid

*tanh activation funcition*
`g(z) = tanh(z)`
`g(z) = (e^z - e^-z)/(e^z + e^-z)`
`g'(z) = 1 - (tanh(z))^2)`- derivation of tanh
`g'(z) = 1 - a^2)`- derivation of tanh

*ReLU activation function*
`g(z) = max(0,z)`
`g'(z) = 0 iz z<0, 1 if z>= 0`

*Leaky ReLU activation function*
`g(z) = max(0.01z,z)`
`g'(z) = 0.01 iz z<0, 1 if z>= 0`


*Gradient Descent*
Forward Propagation
`Z[1] = W[1]X + b[1]`
`A[1] = g[1](Z[1])`

`Z[2] = W[2]A[1] + b[2]`
`A[2] = g[2](Z[2]) = σ(Z[2])`

Back Propagation (Derivatives):
`∂Z[2] = A[2] - Y`
`∂w[2] = 1/m ∂Z[2] A[1]ᵀ`
`∂b[2] = 1/m np.sum(∂Z[2], axis = 1, keepdims = True)`

`∂Z[1] = W[2]ᵀ ∂Z[2] * g[1](Z[1])`, remember * is element-wise product
`∂w[1] = 1/m * ∂Z[1] Xᵀ`
`∂b[1] = 1/m np.sum(∂Z[1], axis = 1, keepdims = True)`

*Random Initialisation*
Weights shouldn't be initialised to 0, doing so will mean that the weights will all have the same value.
Use `np.random.randn(m[l],m[l-1]) * 0.01`.  Multiply by 0.01 to keep values small and closer to the steep part of the graph so that gradient descent is faster when using sigmoid or tanh with binary classification.
Bias can be 0 `np.zero((m[l],1))`.

## Deep Neutral Networks
*Matrix Dimensions*
`z[1] = w[1]x + b[1]`
`z[1] : (n[1], n[0]) + (n[1],1)`

`w[l] : (n[l], n[l-1]) `
`b[l] : (n[1],1)`

`Z[l], A[l] : (n[l], m)`
`∂Z[l], ∂A[l] : n[l], m)`

n[0] = number of features
n[l] = number of nodes in a layer l

*Forwards and Backwards Propagation*
Cache Z during forwards propagation for use in backwards propagation later. While the activation values` A = g(Z))` are passed onto the next layer, Z needs to be cached for use during the derivatives calculations.

Forwards:
`Z[l] = W[l]A[l-1] + b[l]`
`A[l] = g[l](Z[l])`

Backwards:
`∂Z[l] = ∂A[l]*g[l](Z[l])`
`∂W[l] = 1/m ∂Z[l] A[l-1]ᵀ`
`∂b[l] = 1/m np.sum(∂Z[l], axis = 1, keepdims = True)`
`∂A[l-1] = W[l]ᵀ ∂Z[l]`

*Hyperparameters*
Parameters are `W` and `b`.

Hyperparameters includes:
* `α`: learning rate
* # of iterations
* `L`: # hidden layers
* `n[l]`: hidden units
* choice of activation function
