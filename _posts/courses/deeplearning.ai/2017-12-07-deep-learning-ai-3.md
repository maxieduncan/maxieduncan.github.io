---
layout: post
category : course
title: "Deeplearning.ai Notes: Structuring Machine Learning Projects"
tags : [course, notes, ml]
---
{% include JB/setup %}

My notes from: https://www.coursera.org/learn/machine-learning-projects

## ML Strategy
Chain of assumptions:
* Fit training set well on cost function
	* Bigger network
	* Try different optimisation function
* Fit dev set well on cost function
	* Add Regularisation
	* Bigger training set
* Fit test set well on cost function
	* Bigger dev set
* Performs well in real world
	* Change dev set
	* Change cost function

*Single number evaluation metric*
Makes it much easy to assess if things are working and make comparisons.

Precision: out of the predictions made for a class, what percentage where correct e.g. out of the results classified X, P% were correct
Recall: out of the actual classifications for a class, what percent were correctly classified e.g. R% of class X were classified as X

F1 score = `2/ (1/P + 1/R)`, average of P and R

*Satisficing & Optimising metric*
Could combine Accuracy and Running time e.g. `cost = accuracy - 0.5*running time` though this is a bit artificial. Could instead set a limit e.g.  maximise accuracy within a certain running time. Optimising accuracy while satisficing running time.

*Train, dev, test sets*
Choose from the same distribution.
Up to 10,000 training samples, can use a 70|30 or 60|20|20 split. Over that use 1% for dev and test sets.

*Bayes Optimal Error*
Limit at which it becomes impossible to generate better predictions. Accuracy doesn't improve beyond this limit. Theoretical best function.
Can use human level error as an approximation for Bayes error.

Avoidable Bias: if training set error is far off human (Bayes) error, focus on bias reduction techniques;
Variance: if training set error is close to human (Bayes) error but dev set error is larger, focus on variance reduction techniques.

*Surpassing human level*
Good at structured data, not so good at natural perception. Good at lots of data.
* Online advertising
* Product recommendations
* Logistics (predicting transit time)
* Loan approvals

Even getting good at natural perception tasks:
* NLP
* Some image recognition

*Improving your models performance*
1. You can fit the training set well, avoidable bias.
	* Train a bigger model
	* Train longer/better optimisation algorithms: momentum, RMSprop, Adam
	* NN architecture/hyperparameters search
2. The training set performance generalises  well to the dev/test sets, variance
	* More data
	* Regularisation: L2, dropout, data augmentation
	* NN architecture/hyperparameters search

### Error Anlaysis
Review the errors (mislabeled examples, both positive and negative) to determine what issues are best worth investigating.
Evaluate multiple ideas in parallel.

*Cleaning incorrectly labeled data*
DL algorithms are quite robust to random errors in the data set. They are less robust to systematic errors.

During testing against he dev set, record incorrectly labeled results and review that they were labelled correctly.
Review:
* Overall dev set error
* Errors due to incorrect labels
* Errors due to other causes
If the errors due to incorrectly labeled results are due to incorrect labels then it's worth rectifying.

Consider examining examples your algorithm got right as well as ones it got right. It may have got it "right" even though it was incorrectly labeled.

*Build quickly, iterate*
If tackling a new problem:
* Set up dev/test set and metric
* Build initial system quickly
* Use Bias/Variance analysis & Error analysis to prioritise next steps

### Mismatched training and dev/test set

*Training and test distributions that differ*
If adding a new data source that has limited examples, just adding it to your existing large data source won't be affective as only a limited number will end up being part of the dev and test sets.  You'd be better of in this case to skew your data set to spread the new data source more evenly across training, dev and test.

*Estimating Bias and Variance*
The way this is measured changes when you dev and test sets don't come from the same distribution as your training set.
This can result in a bigger difference between the training error and dev error. To avoid this you can create an additional "training-dev" set from the same distribution as the training set, this allows you to perform error analysis on this new set to check for variance issues in your model from the training set by taking the new data out of the equation. Big difference between training and training-dev indicates a variance issue, while a big difference between training-dev and dev indicates a data mismatch issue.

Look for differences in errors between:
* Human (Bayes) -> Training Set = Avoidable Bias
* Training Set -> Training Dev Set = Variance
* Training Dev Set -> Dev Set = Data Mismatch
* Dev Set -> Test Set = Overfitting to dev set

*Addressing data mismatch*
Not great systematic ways to address data mismatch.
Carry out manual error analysis and try to understand the differences between training and dev/test sets.
Make training data more similar; or collect more data similar to dev/test sets.

Artificial data synthesis, manipulating the training data to better reflect the dev/test data. e.g. adding background noise to audio. Risk of overfitting to artificial scenario.

### Learning from Multiple Tasks

*Transfer Learning*
Using part of a trained Neural Network to help solve another task. To do this make use of the earlier layers, retraining the later layers with the new data set. Often referred to pre-training and fine-tuning.

Makes sense when you have a lot of data from the problem you're transferring from but not much where you're training to.
* Task A and Task B have the same input x e.g. (image).
* You have a lot more data for A than B.
* Low level features from A could be helpful for learning B.

*Multi-task Learning*
Rather than assigning a single label to an input, you assign them multiple labels. To this you can use logistic regression but don't use softmax at the last layer.

This could be done by creating multiple Neural Networks but it's more efficient if it can be done by training a single model to produce multiple labels.
* training on a set of tasks that could benefit from having shared low level features
* the amount of data for each task is similar
* can train a big enough neural network to do well on all the tasks

### End-to-end Deep Learning
Requires a large dataset. Removes the need for intermediate steps.
Excludes ability to add hand designed components which can allow humans to inject knowledge (though this can be a double edged sword that doesn't allow new patterns to be found).
