---
layout: post
category : courses
title: "Deeplearning.ai Notes: Recurrent Neural Networks"
tags : [course, notes, ml]
---
{% include JB/setup %}

My notes from: https://www.coursera.org/learn/nlp-sequence-models

### Notation
`x<t>`: value of the input sequence at position t
`y<t>`: value of the output sequence at position t
`Tx`: length of the input sequence
`Ty`: length of the output sequence
`x(i)<t>`: value of the input sequence at position t for input i
`Tx(i)`: length of the input sequence i
`||u||2` is the L2 norm (or length) of the vector  u

For NLP use a vocabulary/dictionary and use the indexes to identify the words it contains.
For identifying words using one-hot you use a vector for each word; use a one to flag the position of the word in the dictionary, 0 the rest of the indices. So the size of each of these vectors is the same as the size of the vocabulary/dictionary.

### RNN Model
Why not use a standard network?
* lengths of inputs and outputs can vary
* doesn't share features learned across different positions of text

A RNN when, computing tilmestep inputs, takes the outputs from previous time steps into account. As such only the preceding values have an impact when calculating a value, which is an issue when relevant context comes up later in the sequence. Bi-directional RNNs help resolve this.

`a<1> = g(Waa a<0> + Wax x<1> +ba)`, activation function is often tanh, sometimes ReLU
`yhat<1> = g(Wya a<1> + by)`, may be a different activation function: sigmoid, sometimes softmax, depends on the output

More generically:
`a<t> = g(Waa a<t-1> + Wax x<t> +ba)`
`yhat<t> = g(Wya a<t> + by)`

`a<t> = g(Wa[a<t-1>, x<t>] + ba)`, matrix version, stacks the vectors together
`yhat = g(Wy a<t> +by)`

### Back Propagation through time
`L<t>(yhat<t>, y<t>) = -y<t> log yhat<t> - (1-y<t>) log(1 - yhat<t>)`, logistic regression loss for binary classification, i.e. cross entropy. Loss for a single word at a single position.

`L = sum(L<t>(yhat<t>, y<t>))`, so sum the losses for every timestep. Minimising this loss (gradient descent) is known as *back propagation through time* as it goes through the sequence from right to left.

### Different types of RNNs
The lengths of the inputs and outputs can often differ. E.g. translation, video activity recognition, etc

So far we've only looked at Tx = Ty. This is a *many to many* RNN.

For a *many to one* RNN (e.g. sentiment classification (rating), activity recognition) you only output a value at the final time step.

For a *one to many* RNN (e.g. music generation) you have a single input (or even no input) and output a sequence. So use the outputs from the previous time steps , with no additional inputs, to produce the output for the next time step.

There's also *variable many to many* RNNs where Tx != Ty (e.g. translation). This RNN is made up out of two parts, the *encoder* which takes in the inputs, and the *decoder* which outputs the translation.

There's also one to one but this is essentially just a standard neural network and doesn't need an RNN.

### Language model and sequence generation

Calculates the probability for a sentence based on the inputs. This is useful where the input could be taken in more than one way to determine what the most likely meaning was.

To do this you need a large corpus (body) of sentences. Tokenise this sentence using a vocabulary. Often use an <EOS> token (end of sentence) to explicitly define when a sentence ends. Can also use <UNK> (unknown token) for words that aren't in your vocabulary.

`L<t>(yhat<t>, y<t>) = - sum(y<t> log yhat<t>)`
`L = sum(L<t>(yhat<t>, y<t>))`

### Sampling novel sequences
After training a RNN model as above you then want to be able to sample a sequence.

Starting from zero, use the softmax distribution to predict the first value. This is then input to the second time step to predict the second word and so on. Keep going until <EOS> or if that's not in the vocabulary keep going for a set number of words. May want to remove the <UNK> token from the vocabulary.

/So I guess this allows you to create random sentences?/

You can also use a character vocabulary rather than a word based one. Don't need to worry about unknown word tokens. The main disadvantage is that you have much longer sequences and don't pick up on long term patterns as well.

### Vanishing gradients with RNNs
Words earlier in the sentence can have a large impact on words at the end which the RNN model discussed so far doesn't handle well.

e.g. whether something should be plural:
The cat ... was ...
The cats  ... were ...

The deeper a network the less likely it is that values at the end will propagate all the way back to the start. It's difficult for an output near the end to be influenced by an input near the start of the sequence.

Exploding gradients don't tend to be such a common issue with RNNs and they're easier to identify as the parameters blow up. Can apply gradient clipping to resolve.

### Gated Recurrent Unit (GRU)
Helps with the vanishing gradient descent problem.

`a<t> = g(Wa[a<t-1>, x<t>] + ba)`

An RNN unit takes in the activation values from the previous unit (`a<t-1>`) and along with the inputs for the current unit (`x<t>`) passes them through an activation function (e.g. tanh) producing the activation outputs for the next unit `a<t>` and can also use an output function (e.g. softmax) to produce an output value (`yhat`).

A GRU adds a memory cell to keep track of previous activations.
A candidate for the memory cell value is calculated using:
`c~<t> = g(Wc[c<t-1>, x<t>] + bc)`, tanh for activation function, note memory cell has it's own parameters

When considering a candidate `c~<t>` as the new value for `c<t>`, the game `Gu` is to determine if the candidate value will be used or not.

You calculate a Gate value between 0 and 1 using a sigmoid function.
`Gu = σ(Wu[c<t-1>, x<t>] + bu)`, this will typically be close to 0 or 1 based on Sigmoid s curve.
While the gate is 0 (or close to) then the value of the memory cell is essentially maintained until it's close to 1 which means the value is no longer required. Because the value of Gu is close to 0 most of the time it helps avoid the vanishing gradient problem.

So:
`c<t> = Gu * c~<t> + (1-Gu)*c<t-1>`

In GRU the memory cell is also the activation unit, i.e. `c<t> = a<t>`.

`c<t>` can also be a vector allowing multiple values to be stored. In this case `Gu` and `c~<t>` will also be the same dimension.


#### Full GRU
Adds `Gr`:
`c~<t> = g(Wc[Gr * c<t-1>, x<t>] + bc)`
`Gr = σ(Wr[c<t-1>, x<t>] + br)`

#### Notation
In other writings you may find the notation referred to as:
`h~ = c~<t>`
`u = Gu`
`r = Gr`
`h = c<t>`

GRUs have allowed RNNs to capture and track long term features making them more effective.

### LSTM Unit
The Long Short Term Memory unit is a more powerful and general version of the GRU. In LSTM the activation unit and memory cell are no longer the same `c<t> != a<t>`.

`c~<t> = g(Wc[a<t-1>, x<t>] + bc)`, note the use of the activation unit here rather than the memory cell
`Gu = σ(Wu[a<t-1>, x<t>] + bu)`, update gate
`Gf = σ(Wf[a<t-1>, x<t>] + bf)`, forget gate, replace an instance of `Gu` when calculating c<t>
`Go = σ(Wo[a<t-1>, x<t>] + bo)`, output gate
Now `c<t> = Gu * c~<t> + Gf*c<t-1>` compared to: `c<t> = Gu * c~<t> + (1-Gu)*c<t-1>` in GRU.
`a<t> = Go * c<t>` note slide also contains version: `a<t> = Go * tanh c<t>`, i.e. applies activation function `a<t> = Go * g(c<t>)`

So LSTM is slightly more complex and places the gets in different places.

#### Peephole connection
Adds c<t-1> to the gates. First element affects first gate, second the second etc.

#### Which to use?
Hard to say. LSTMs came before GRUs. Debatable as to which is best. GRUs are simple but LSTM more powerful. Most people default for LSTM though GRU has been gaining momentum.

### Bidirectional RNN
Adds activation until going in the reverse direction of the sequence. This is not backwards propagation, it's forwards propagation but with the input sequence reversed. The combination of the two activations for each unit are then used to make the predictions.

`yhat<t> = g(Wy[a-><t>, a<-<t>] + by)`

So this takes into account data from both the past and the future. The units used can also be GRU or LSTM.

The disadvantage is that you need to entire sequence before you can run the calculation. So for real time speech there are more complex models that can be used.

### Deep RNNs
Adds layers to a RNN `a[l]<t>` which are then stacked to produce the output. So activation layers have values coming the previous units and the previous layers.
`a[l]<t> = g(Wa[l] [a[l][t-1], a[l-1]<t>] +ba[l]`

Typically you don't see a huge number of layers. Complexity of more than a few layers becomes unmanageable.

Can also have deep layers at the output layer that aren't connected horizontally, i.e. only applied to the RNN output.


## NLP & Word Embeddings
### Word Representation
So far we've used a vocabulary and a one-hot representation. Weakness is that it treats each unto itself and doesn't support associations. Word embedding allows a featurised representation, given features rates words on how well they match that feature. The closer the words are in this multi dimensional space the more related they are.

T-SNE provides a way to represent this information in a 2D visualisation.

Allows you to apply transfer learning. Can train on large corpuses of text from the internet (or download pre-trained embeddings) and use on smaller training set. Instead of using a sparse one-hot vector can use a dense multi dimensional vector for training. Can continue to fine-tune with new data.

### Properties of Word Embeddings
Can help with analogy reasoning.

`e_man - e_woman ≈ e_king - e_w` - > `similar(e_w, e_king - e_man + e_woman)`, find a word to match `e_w` .
Most commonly used similarity function is Cosine similarity. `sim(u, v) = u.T v / ||u||2 ||v||2`. Calculates the angle between the two points.
Can also use squared distance though not as popular. Main difference is how it normalises.

### Embedding matrix
Shaped as number of embeddings x size of vocabulary.

Multiply by the one-hot vector to get the features for a particular word. In practice, use specialised function to look up an embedding which is more efficient rather than determine this by matrix vector multiplication.

### Learning word embeddings
To predict the next word in a sentence put the embeddings for each word through a hidden layer then a softmax layer to predict the next word. This will learn pretty good word embeddings. Often use a window, e.g. the previous four words, rather than the entire sentence. For context may take words on the left and right for the window. Can also use last 1 word, or nearby 1 word. Result in meaningful word embeddings.

### Word2Vec
Predict a *target* word given a *context* word. Skip-gram model. Suffers from slow computational speed. Hierarchical softmax classifier.
Typical use methods when selecting the context words to sample that reduce selection of the more common words to get a more balanced selection.

### Negative Sampling
Pick a context word and a target word. Label as 1. Pick random target words from the dictionary and label them as 0. It's OK if by chance if a valid association is randomly chosen. For smaller datasets the random target words should be 5-20, for larger 2-5.

More efficient than Word2Vec. Called negative sampling.

### GloVe word vectors
Global vectors for word representation.
Number of times words appear in close proximity to each other. Done my minimising square cost function.

### Sentiment Classification
Use word embeddings to predict sentiment, e.g. rating.

Word order is important, can loose context by just looking for positive/negative words. "Lacking good", "not great", "nothing bad".

Feed the word embeddings into a many to one RNN. Takes word sequence into account.

### Debiasing word embeddings
Removing racism, sexism, etc.

To identify take related terms and difference them then take the average.
Some words are naturally gender biased, e.g. grandmother and grandfather. For words that aren't definitional like this, e.g. doctor and nurse, project to remove the bias.
Equalise definitional Paris, e.g. mother and father so that they are equidistant to related terms.

## Sequence models & Attention mechanism
### Basic Models
#### Sequence to Sequence Model
E.g. translation
Encoder network to take the input, then a decoder network to produce the output.
E.g. Image captioning
Use a CNN to identify features then pass then into a decoder network.

### Picking the most likely sentence
Think of machine translation as building a conditional language model. Machine translation is very similar to the language model, the difference is that it has an encoder network at the start to produce the input for the language model (the encoding network).

This gives you the probability of different translations. You don't want to sample randomly at this point, instead you want the translation with the highest probability using something like beam search. Greedy search, taking the best words in order, isn't the best algorithm for finding the best solution on many translations.

### Beam Search
Whereas greedy search finds the most likely word and continues on, beam search considers multiple possibilities. The beam width is the number of possible choices it will consider at each step.

Having picked the most likely choices for the first word, it then considers choices for the second word using each of these choices. Number of choices considered at each step are beam width by the size of the vocabulary. The number of networks created is the beam width.

### Refinements to Beam Search
#### Length Normalisation
Longer sentences can result in an output probability that is too small to compute accurately given rounding errors from the multiplication of many probabilities that are smaller than one.. This means that the algorithm has a tendency to produce short sentences over longer ones. Using a log function on the probabilities and summing the results helps avoid this when maximising the probabilities.

At the end you pick the output that comes up with the highest probability.

#### Beam Width
The wider the beam the more choices that are considered but the more processing that is required. In production you'll often a beam width of 10; 100 is considered larger; 1000s are used in research. You get diminishing returns as the beam width gets wider. Beam width of one is essentially greedy search. Beam search is not guaranteed to find the optimal solution.

### Error Analysis in Beam Search
Determining if it's the RNN (the encoder and the decoder) or the Beam Search that's the issue.
Always tempting to just increase the beam width (more data) but not always effective.

With the RNN it can be worth testing a bad result  (`yhat`) against a known good translation (`y*`) and comparing the probabilities.
Case 1: P(y*|x) > P(yhat|x), bean search is at fault.
Case 2: P(y*|x) <= P(yhat|x), RNN is at fault.

After analysing a number of erroneous results you can compare the rate of beam search to RNN errors and using the ratio to determine where to focus improvements.

### Bleu Score (Bilingual Evaluation Understudy)
How do you deal with the case where there are multiple good translations. Bleu gives a score to translations.

### Attention Model Intuition
Working on small part of a sentence at a time rather than processing in one go. Helps with longer sentences.
The attention model uses weights to determine the part of the sentence to consider when translating a word. This allows the translation model to take some context from the sentence into account. Both the activations of the BRNN and state from the previous step are used to determine how much attention is paid to surrounding words in a sentence.

### Attention Model
Use a BRRN or LSTM network.
`a'<t>` is the activations for the forwards and backwards BRNN steps
`s'<t>` forward only RNN state, has a context `c` and depends on attention parameters `α` that specify how much attention should be given surrounding words. The context `c` is a weighted sum of these attention weights applied to the activations `a'<t>`.  `c<t> = sum(α<t, t'> a<t'>)`.
`α<t, t'>` amount of attention `y<t>` should pay to `α<t'>`
`α<t, t'> = exp(e<t,t'>) / sum(exp(e<t,t'>))`
Both `e<t,t'>` and `α<t, t'>` are dependant on `s<t, t'>` and `a'<t>` . So `α<t, t'>` is determined by training a NN using `e<t,t'>` and `α<t, t'>`.

Downside is that it takes quadratic cost to run. This may be OK for smaller sentences but struggles on longer ones.

### Speech Models
Phonemes (writing down the sounds of words) used in the past but advances in deep learning and large data sets means this isn't necessary.

Can use the attention model.

CTC (Connectionist temporal classification) cost for speech recognition. RNN that outputs an output for each input.

100Hz * 10 seconds = 1000 inputs.

Collapse repeated characters not separated by blank (note blank is not space).

### Trigger Word Detection
Set target output to be one when trigger word heard. One negative is that this outputs a lot of zeros making training data imbalanced. This can be mitigated to a certain degree by a hack by adding extra ones after each output.
