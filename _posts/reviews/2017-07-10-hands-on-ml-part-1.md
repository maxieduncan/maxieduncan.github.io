---
layout: post
category : books
title: "Hands-On Machine Learning with Scikit-Learn and TensorFlow: Part 1"
tags : [book, review, notes, ml]
---
{% include JB/setup %}

I picked up [Hands-On Machine Learning with Scikit-Learn and TensorFlow](http://shop.oreilly.com/product/0636920052289.do) by Aurélien Géron recently and these are just my notes from Part 1: The Fundamentals of Machine Learning. These notes don't won't do much more than highlight my own shallow understanding of the material but for me they make a handy reference (although my note taking was sporadic, especially at the beginning).

I did Andrew Ng's [Machine Learning](https://www.coursera.org/learn/machine-learning) course on Cousera earlier in the year (highly recommended) which meant I came in with a reasonable understanding of the concepts and some of the algorithms already; while this certainly helped I do feel that my grip of the underlying mathematics remains dubious.

I did enjoy the hands ons nature of the book and the introduction to Scikit-Learn which has bean on my list to look into for a while. The exercises and being able to run the code yourself certainly helps. Unfortunately I struggled to concentrate through some of the more mathematical sections (especially "Under the Hood" in the SVM chapter) but they're still there for me to refer back to. The high level concepts do make sense to me though and the big benefit for me has been expanding my ML knowledge with new algorithms and a deepening understanding of what can be done.

My notes are grouped by the Chapters and are light on detail, if you want that I'd recommend buying the book!

## Machine Learning Landscape
### Machine Learning Types
#### Supervised learning: labeled data
* Classification: labelling new data
* Regression: predicting values

#### Unsupervised learning: unlabelled data
* Clustering, identifying data grouped by common features
* Visualisation of data
* Anomaly detection, finding outliers in the data
* Dimensionality reduction, feature extraction
* Association rule learning, discovering associations between attributes

#### Reinforcement learning
Uses an Agent that observes the environment and select then perform actions. Reward and punishment for actions taken are used as feedback to improve further selection.

### Batch and Online learning
#### Batch/Offline
* Lots of data, trained in one go then the solution is deployed

#### Online
* Incremental, training is done in steps, training can be continued when the solution is deployed
* Learning rate: too fast lose old data, too slow don't respond fast enough to changes

### Instance based vs Model based
#### Instance based
The system learns examples by heart, then generalises new cases using a similarity measure

#### Model based
Build a model of the examples, then use those model to make predictions
* utility/fitness function: how good the fit is
* cost function: how bad the fit is

### Fitting the model
#### Overfitting
If model performs well on training data but poorly on test

#### Underfitting
Performs poorly on both training and test data


## Classification
Train a model to predict what class an input belongs to. Typically linear models are used for this, but regression models can be used as well.
* Binary, choose between two classes
* Multi class, distinguishes between two or more classes
* Multi label, predict multiple classes for inputs (e.g. face labelling in a photo)
* Multi output, rather than just outputting the predicted class, outputs the predictions for each one, for each input.

### Performance Measures
#### Cross Validation
Splitting the dataset into a training set and a test set. The training set is used to train the model, the test set to evaluate it's performance on data it hasn't seen during training.
K-Fold Cross Validation is used to split the training set into randomly partitioned sub sets for training.

#### Confusion Matrix
The confusion matrix provides a comparison between predicted and actual results.
```
              | Predicted No    | Predicted Yes   |
Actual No     | true negatives  | false positives |
Actual Yes    | false negatives | true positives  |
```

#### Precision/Recall
The Confusion Matrix can be used to calculate Precision and Recall.
*Precision*, the accuracy of positive predictions: TP / TP +FP
*Recall* (Sensitivity, True Positive Rate), the ratio of positive instances correctly labelled: TP / TP + FN
*F-Score*: F = (P * R)/(P + R) * 2
The F score favours similar precision and recall, in some cases it can be more important to have one over the other.

#### ROC Curve
Receiver Operating Characteristic Curve, plots the Recall (True Positive Rate) against the False Positive Rate.
Area Under the Curve, 1 = perfect, 0.5 = random

## Linear Modelling Algorithms
### Normal Equation
Mathematical equation that gives the direct result.
* Slow with large number of features
* Fast to process large data set, requires to load everything into memory

### Gradient Descent
Generic optimisation algorithm, tweaks parameters to minimise the cost function.
* Suffers if the features aren't normalised
* Can get confused by local minimums
* If learning rate is too small can take an excessive time, if too big can miss the global minimum entirely.

#### Batch Gradient Descent
* Slow on large data sets
* Fast on large number of features

#### Stochastic (Random) Gradient Descent
* Relies on randomness during training, set the random_state parameter to get consistent results
* Can handle large datasets efficiently
* Deals with training instances independently, so good for online learning

#### Mini-batch Gradient Descent
Combines Batch and Stochastic by randomly selecting small batches to train against.

### Regularised Linear Models
* Ridge Regression, adds a regularisation term to the cost function, keeps the weights small
* Lasso Regression, tends to eliminate the weights of the least important features (set to 0). Essentially automatic feature selection.
* Elastic Net, mix of Ridge and Lasso Regression. When r=0, Elastic Net is equivalent to Ridge Regression, when r=1 it's Lasso Regression

So which to use? You generally want some regularisation over a pure linear model and Ridge makes a good default. If only a few features are actually useful then Lasso or Elastic Net (though note that Lasso becomes unreliable if the number of features exceeds the number of training instances, or many of the features have a high correlation)

### Logistic Regression
If only two outputs then it's a binary model that can be used for classification i.e. if probabilities > 50% belongs to that class, if not then the other.

So it estimates probabilities and makes predictions. This is done by training it to set the parameters theta so that the models estimates his probabilities for positive instances (y=1) and low for negative instances (y=0).

There is no known closed-form equation to minimise the cost so Gradient Descent is used. As the cost function is normalised GD will never get stuck in a local minimum.

### Softmax Regression
Used for multi classification. Each class has it's own set of parameter vector theta.

It is multi class, NOT multi output.

## Support Vector Machines
Linear and non-linear classification, regression and outlier detection. Particularly good for complex small-medium sized classification.

### Linear SVM Classification
The intend is to create the widest "street" possible between the data sets. This isn't always possible so then you need to balance widening this gap and reducing the margin violations where outliers from one set show up on the wrong side.

Unlike Logistic Regression SVM classifiers don't output probabilities for each class, just the predicted class.

### Non Linear SVM Classification
Use polynomial features to make a non linear dataset linear.

Low polynomial degree can't deal with complex data sets but a high polynomial degree creates a large number of features reducing performance.

#### Adding Similarity Features
Use a similarity function like the Gaussian RBF (Radial Basis Function)  that measures how much each instance resembles a particular "landmark".  The simplest approach to choosing the landmark is to create one for each input though this doesn't scale well for large data sets.

### SVM Regression
SVM supports both linear and non linear regression. Instead of trying to create the widest street between classes, try to fit as many from the same class as possible while keeping margin violations down.

A /support vector/ is any instance that ends up on the "street", including the margins. Computing the predictions involves only the support vectors, not the whole training set.

Scaling is important as otherwise the smaller features will have less impact than the features with larger values.

## Decision Trees
Used for classification, regression and multi output tasks. Fundamental components of Random Forests.

Don't require much in the way of data prep. Don't require feature scaling or entering at all.

Unfortunately finding the optimal solution is an NP-Complete problem.

Prediction is fast but training is slow.

Decision trees have a tendency to overfit the data if not regularised (e.g. limiting the max depth).

They like orthogonal decision boundaries, which makes them sensitive to training set rotation. You can use PCA to get a better orientation of the training data.

Very sensitive to small variations in the training data though this can be overcome by the use of Random Forests which emerge predictions over many trees.

Pre-sorting training data can speed up the training if there's only a few thousand instances, it will slow it down as the data increases.

#### CART
Classification And Regression Tree, a greedy algorithm that looks for the best split at each step meaning that it won'r necessarily find the optimal solution.

#### Regression
The predicted value is the average of the values in the region.

## Ensemble Learning & Random Forests
You typically use an ensemble of models at the end of a project when you have a few good models trained.

A Random Forest is an ensemble of Decision Trees.

Ensemble methods work best when the predictors are as independent as possible. This can be helped by using different algorithms.

Hard vs Soft Ensemble: hard just counts the votes for a class, soft uses the probability.

### Bagging
Known as Bagging (Bootstrapping) when sampling is performed with replacement, (Pasting) when without. Allows training instances to be sampled several times across multiple predictors (Bootstrapping allows training instances to be sampled several times for the same predictor).

Bootstrapping adds a bit more diversity to subsets but can introduce more bias but also means that the predictors end up less correlated so the ensembles variance is reduced. Typically Bootstrapping provides better results.

When Bootstrapping only about 67% of the training set will be used when training a predictor, meaning that the unused portion of the set for each predictor (note the sets aren't shared for each predictor) can be tracked and used for testing (known as Out-of-Bag evaluation).

##### Random Patches
Sampling Bothe training instances and features.

##### Random Subspaces
Only sampling the features, using all the training instances.

### Random Forests
An ensemble of Decision Trees. A random subset of features is used at each node with the Decision Tree searching for the best possible thresholds.

##### Extra-Trees (Extremely Randomised Trees)
Randomises the thresholds of each feature acting as a form of regularisation.
As they're not searching for the best thresholds they're faster to train though take the same time to predict.

#### Feature Importance
The features with the most impact tend to show up at the top of the tree, so taking an average of the depth across all trees in the forest can give you some insights.

### Boosting
Ensemble method that combines several weaker methods into a stronger one.

Because of the sequential nature of this technique it cannot be parallelised.

#### AdaBoost
Focusses training new predictors on the instances that the previous models have underfit by weighting the training instances. When combined each predictor is weighted based on it's accuracy.

If overfitting you can try reducing the number of estimators or regularising the base estimator.

#### Gradient Boosting
Similar to AdaBoost as it's sequential but uses the /residual errors/ (the difference between the expected outputs and the actual) of the previous model to fit the new predictor instead of tweaking the instance weights.

You can use early stopping to help prevent Gradient Boosting from over fitting. Decrease the learning rate to help over fitting.

### Stacking
Different models are trained on a partial training set. The training data not used is then used to created predictions using these trained models. A blender is trained on these predictions.

You can add an extra layer by splitting the training data into three, then using different models in each blender to feed into a final blender.

## Dimensionality Reduction
More features can increase the accuracy of the predictions but slow training of the model. Reducing dimensionality reduces accuracy but speeds up training. Can also help for visualisation.

### Projection
Reducing dimensions. Many features are either nearly constant or highly correlated. By simply projecting onto a plain these features can be removed speeding up the training time.

### Manifold Learning
A /d/-dimensional manifold is part of an /n/-dimensional space (d<n) that locally resembles a /d/-dimensional hyperplane. The Swiss roll example is an example of a 2D /manifold/ that is rolled out of a 3D space.

The /manifold hypothesis/ says that many high dimensional data sets lie close to lower dimensions manifolds, something that's often true in practice.

You can think of it that many features in a training set will contain a rather limited range of values compared to those that are theoretically possible.

There's an assumption in doing this that the lower dimension manifold will be easier to model, this isn't always true however.

### PCA
Principal Component Analysis is the most popular dimension reduction algorithm. It identifies the hyperplane that lies closest to the data then project onto it. Requires all the data to be loaded into memory (Incremental PCA avoids this).

You can reverse the process but because of the lost features (due to the reduced dimensions) you'll loose accuracy.

##### Preserving the Variance
The hyperplane is selected by measuring the resulting variance and maximising it, therefore likely loosing the least information.

##### Principal Components
PCA identifies the axis that provides the largest amount of variance in the training set, this is done by minimising the mean squared distance between the data and it's projection onto the axis.

##### SVD
Singular Value Decomposition allows you to find the Principal Components of a data set (their is one for each dimension).

##### Projecting down to d Dimensions
Once the Principal Components have been identified, you can reduce the data set down to /d/ dimensions by projecting on to the hyperplane defined by the first /d/ dimensions identified by SVD. This ensures the most variance as possible is retained when reducing to /d/ dimensions.

#### Incremental PCA
Doesn't require the entire training set to be loaded into memory. Split the training set into mini batches, good for large training sets and online learning, slower.

#### Randomised PCA
Uses a stochastic algorithm to find an appreciation of the Principal Components. Dramatically faster when /d/ is much smaller than /n/.

#### Kernel PCA
An unsupervised learning algorithm, uses Kernel techniques to perform complex NonLinear projections for dimensionality reduction.  Good for clustering.

### LLE
Locally Linear Embedding, another very powerful NonLinear Dimensionality Reduction. A Manifold Learning technique that doesn't rely on projections. Good for unrolling twisted manifolds. Scales poorly for large datasets.

Measures how close each instance is linearly to it's closest neighbours, then looking for a low dimension representation that preserves these relationships.

### Combined Algorithms
You could use PCA to reduce the dimensions, then LLE. This is likely to get the same results as just using LLE but much faster.
