---
layout: post
category : courses
title: "Deeplearning.ai Notes: Convolutional Neural Networks"
tags : [course, notes, ml]
---
{% include JB/setup %}

My notes from: https://www.coursera.org/learn/convolutional-neural-networks

## Notation
f[l] = filter size
p[l] = padding
s[l] = stride
nc[l] = number of filters

input: `nh[l-1] x nw[l-1] x nc[l-1]`
output: `nh[l] x nw[l] x nc[l]`
filter: `f[l] x f[l] x nc[l-1]`
activations: `nh[l] x nw[l] x nc[l]`
weights: `f[l] x f[l] x nc[l-1] x nc[l]`, filter x number in current layer
bias: `nc[l]`, `(1, 1, 1, nc[l])`

`nh[l] = floor( (nh[l-1] +2p[l] - f[l])/s[l] +1 )`, p i padding
`nw[l] = floor( (nw[l-1] +2p[l] - f[l])/s[l] +1 )`

## Convolutional Neural Networks
Used for computer vision.
* Image classification
* Object detection
* Neural Style Transfer, combine two images to create a new image

Uses a "filter" to process subsets of the input matrix.

*Edge Detection*
Convolution using a vertical 1,0,-1 matrix allows vertical edges to be identified.
```
1,0,-1
1,0,-1
1,0,-1
```
You get positive or negative results depending on if going to dark to light or vice versa; taking the absolute will allow you to just identify edges.
A horizontal 1,0,-1 matrix allows horizontal edges to be identified.
```
1,1,1
0,0,0
-1,-1,-1
```

The numbers in this matrix can be treated as parameters; other values than 1,0,-1 can be used in the matrix and it's best to learn them from the data to identify all sorts of features.

*Padding*
For an `n x n` image and an `f x f` image, the output will be: `n-f+1 x n-f+1`. This means that the image will shrink the filter is applied. It also means that the pixels along the edges don't get used as much.
To solve both shrinking output and throwing a lot of the information on the edges, you can pad the image by adding zeros along the border. Now the equation becomes `n+2p-f+1 x n+2p-f+1`, which means that the image retains it size.

How much to pad?
"Valid" i.e. no padding : `n x n`, * `f x f ` -> `n-f+1 x n-f+1`, smaller output
"Same" the output is the same as the input: `n+2p-f+1 x n+2p-f+1`, where `p = (f-1)/2`
Note: f is almost always odd to keep the padding as a whole number.

*Strided Convolutions*
Stride means that move the filter across by `s` steps rather than 1.
Now output is: `(n+2p-f)/s +1 x (n+2p-f)/s +1`, use the floor incase your filter and step make would extend beyond the border, i.e. isn't a whole number.
Known in mathematics as cross-correlation.

*Convolutions over Volume*
Will generate a 2D image. So the RGB channels will be converted i.e. a 64x64x3 image, will have a 3x3x3 filter and generate a 4x4 image (no padding). Means that with a 3x3 filter you have 27 rather than 9 additions to perform.
To ignore a channel layer, you can zero it.
By stacking multiple filters, you can output a 3 dimensional matrix.

*One Layer of a Convolutional Network*
Add bias to the convolution then apply something like ReLU. Each filter has it's own bias.

*Pooling Layers*
Max Pooling: take only the max value from a filtered region and send to the next layer. So size of next layer will always be smaller.
Average Pooling: take the average in each filter

No parameters to learn, just hyperparameters to tune.

Pooling layers typically aren't counted as layers but can be treated individually. Pooling layers typically follow a Convolutional layer.

*Why Convolutions*
Reduces the size of the input data.

`#parameters = f[l] x f[l] x nc[1] + nc[l]`

Parameter sharing: a feature detector (i.e. a filter, such as a vertical edge detector) will likely be useful if applied to other areas of the image.

Sparsity of connections: In each layer, each output value depends only on a small number of inputs (the ones the filter was applied to).

## Case Studies
### Classic Networks
*LeNet - 5*
Goal to recognise hand written digits. In the 90s, small greyscale images. Used average rather than max pooling, sigmoid/tanh.
*AlexNet*
Competed in ImageNet Large Scale Visual Recognition Challenge in 2012 where it performed much better than other competitors. Similar to LeNet but much bigger, made use of ReLU over sigmoid/tanh, multiple GPUs, local response normalisation.
*VGG - 16*
Just 3x3 filters with stride 1 and same padding and, max pool 2x2 with stride 2. Essentially doubles the filter size every layer.

### ResNets
Skipped Connections: skipping layers to connect earlier activations with layers deeper in the network.
Residual Block: adds activations from earlier layers to the activation function, i.e. a[l+2] = g(z[l+2] + a[l]. So the earlier activations are added after the linear function (X) but before the activation function (g, such as ReLU).

ResNets are made up out of stacked Residual Blocks. This helps with vanishing/exploding gradient issues and allows much deeper networks to be trained.

Adding the additional addition doesn't hurt performance.

The dimensions of the convolutions typically use 'same' padding as the matrices will be added, though weights can be applied to reshape the earlier activation matrix to reshape it (e.g. after pooling).

### 1x1 Convolutions
If you have only one channel it's just multiplication, when you have more than one channel and multiply it by a 1x1xnc, you go to a 1x1xnf matrix; i.e. you go from c channels deep to f filters deep.

So while pooling will shrink height and width, 1x1 convolutions can be used to shrink the number of channels.

### Inception Network
Apply multiple filter sizes using same padding and/or pooling (with padding), and stack the results into a single output.

This does have a high computational cost. Number of operations is: `(nh x nw x #filters) x (f x f x nc)` .
Using 1x1 convolutions you can reduce the cost by a factor of 10 by reducing the number of channels.
/How do you choose the number of filters in the 1x1 convolutional layer?/

Can add side branches to produce outputs at different levels.

Stack Inception modules to create an Inception Network.

## Practical advice for using ConvNets
Look for existing open source examples.

*Transfer Learning*
If you have a small dataset and can make use of an existing NN already trained on relevant data, you can use transfer learning to take the existing model, freeze it then add and train a new softmax layer with your data.

Potentially you can take a copy of the activations in the last layer and just train the parameters for your new softmax layer with your data. This means that you don't need to recompute your epochs for already trained data.

If you don't get good matches then you can start unfreezing deeper layers. Either retrain the weights or replace with new layers. Could even start with the supplied weights as a starting point and retrain the entire network.

Seriously consider using unless you have a lot of data and a lot of computational power.

*Data Augmentation*
Especially useful with images.

* Mirroring
* Random cropping
* Rotation, Shearing (e.g. lean vertical axis),  Local warping, e.t.c. Less used
* Colour shifting (+/- RGB channels), PCA Colour Augmentation

To save disk space you can distort the images as they are loaded by processing the stream as they're loaded in separate threads allowing you to run the training in parallel.

*State of Computer Vision*
More data means you can get away with simpler algorithms and less hand engineering. When you have less data then manual hacks and more complex algorithms are needed.

Tips for benchmarks (not typically for production use):
* Ensembling: train several networks independently and average their outputs (not their weights).
* Multi-Crop at test time: run classifier on multiple versions of test images and average results. 10 crop: take image, corner crops, flip and repeat.

*Use Open Source Code*
* Use architectures of networks published in literature
* Use open source implementations if possible
* Use pretrained models and fine-tune on your dataset.

## Object Detection
### Detection algorithms
*Object Localisation*
Identifying the part of the image that contains the object being classified. To this add four outputs that identify the bounding box of the object: `bx, by, bw, bh`.

*Landmark Detection*
Just need to output coordinates: `lx, ly`. Can add coordinate outputs for multiple landmarks.

*Object Detection*
Identifying multiple objects in an image.
Train on cropped images, then use sliding windows detection.

*Sliding Windows Detection*
Shift different sized windows across the image using the ConvNet you've trained on the cropped images to identify if the window contains an object or not.
Has a big computational cost. Using a big stride eases computational cost but hurts performance making it less likely to successfully identify object.

*Convolutional Implementation of Sliding Windows*
A convolutional implementation greatly reduces the computational cost of Sliding Windows Detection.
Rather than passing individual windows into the ConvNet one at a time, you run the ConvNet on the whole image and the output will contain the results for all windows (rather than just a single output). This shares a lot of the computation.

*Bounding Box Predictions*
YOLO: You Only Look Once. Split image up into a grid and apply the ConvNet to each cell. Works if only one object in the grid cell. The midpoint of an object is used to determine which grid cell it belongs to. The more cells in the grid, the lower the likelihood of more than one object showing up in a cell. It's a Convolutional implementation so runs fast and reduces the amount of computation required.
The x and y outputs will be between 0 and 1, while the h and w params will be above 0 and potentially above 1 if they exceed the cell dimensions.

*Intersection Over Union*
Compares the boundary box size that is predicted to the actual boundary. Divides the intersection (area where they overlap) by the union (the combined area of both boundaries). Considered correct if IoU >= 0.5.

*Non-max Suppression*
Makes sure that your algorithm detects each object only one.
Use `pc` (probability of match) to select the boundary with the highest probably and remove any other matches that it significantly overlaps. Repeat.
* Discard matches where pc <= 0.6
* While there are remaining boxes:
	* Pick the box with the largest pc, output as prediction
	* Discard any remaining boxes with IoU with the box output in the previous step
If you have multiple output classes, run non-max suppression for each one.

*Anchor Boxes*
Allows overlapping objects to be output by adding "anchor boxes" to the output, which allow multiple object outputs to be included in the output for a cell. The output box that a match is added to is determined by calculating the IoU of the match and the anchor box.

Doesn't handle having more objects than anchor boxes in a cell or multiple matches aligning with one anchor box.

*Region Proposals*
R-CNN, identifies regions that look like they might have content. Only these regions are processed. Output label + bounding box.
Fast R-CNN: uses convolution implementation of sliding windows.
Faster R-CNN: Uses convolutional network to propose regions.

R-CNN is currently slower than YOLO.

## Face Recognition & Neural Style Transfer
### Face Recognition
Verification:
* Input image, name/ID
* Output whether the input image is that of the claimed person
Recognition:
* Has a DB of K persons
* Get an input image
* Output ID if the image is any of the K persons (or "not recognised")

*One Shot Learning*
Need to be able to recognise a person given one single image of a person for training as you typically only have one picture of a person.
Learn a "similarity" functions: d(img1, img2) = degree of difference between images. This means that you don't need to retrain a CNN when people leave or join. Instead you check the image using the stored image and see if it meets a threshold of similarity that means you consider them to be the person in the DB.

*Siamese Network*
Instead of feeding a CNN into a softmax function at the end you output a vector of numbers. You then compare this output with the output generated for stored images and compare. `d(x1, x2) = ||f(x2) - f(x2)||^2`. Learn parameters so this difference is small for the same person, of large if they're different.

*Triplet Loss*
Use an Anchor image and compare with a Positive and a Negative example to compare (so three images).
`||f(A)-f(P)||^2 - ||f(A)-f(N)||^2 + margin <= 0`, margin is used to ensure you don't train a CNN to just output a constant like 0.
`Loss(A,P,N) = max(||f(A)-f(P)||^2 - ||f(A)-f(N)||^2 + margin, 0)`
`J = sum(Loss(A,P,N))`

When training if the Negative images are chosen randomly you don't learn as much. You really want to train against similar looking negatives.

*Face Verification and Binary Classification*
An alternative to triplet loss is to pass the difference between the image vector outputs (encodings) into a logistic regression function and train the loss on the output.
e.g. `yhat = σ(sum |f(x1) - f(x2)| + b)`

To improve performance you can store the encoding of the anchor images rather than calculating it every time.

### Neural Style Transfer
Recreating an image in the style of another.
C = Content image
S = Style image
G = Generated image

*Cost Function*
`J(G) = αJcontent(C,G) + βJstyle(S,G)`, hyperparameters to weight the the content and style costs

Find the generated image G
1. Initiate G randomly.
2. Use gradient descent to minimise J(G)
G = G - (∂/∂G)J(G)

*Content Cost Function*
* Use a hidden layer l to compute content cost. (l is typically somewhere in the middle of you CNN)
* Use pre-trained ConvNet
* Let `a[l](C)` and `a[l](G)` be the activation layer l on the images
* If `a[l](C)` and `a[l](G)` are similar, both images have similar content
`Jcontent(C,G) = 1/2||a[l](C) - a[l](G)||^2`

*Style Cost Function*
Use a layer l's activation to measure "style".
Define style as correlation between activations across channels. This means when looking at the channels in a layer, you look at how close the activations are. If channels are highly correlated it means that the features they identify are commonly output together, i.e. represent a style.

* Let a[l] h,w,c = activation at (h,w,c). G[l] is shaped `nc[l] x nc[l]`
* `G[l](c1,c2) = sumh(sumw(a[l](c1) a[l](c2)))`, sum up the multiplication of the activations for the two channels in the style image.
* Calculate this for both the Style and the Generated image.
* `Jstyle[l](S,G) = 1/(2 nh[l] nw[l] nc[l])^2 sumc1(sumc2( G[l](S) - G[l](G) ))`
* Sum for all layers to calculate `Jstyle(S,G)`
* Have a weight hyperparameter for each layer

*1D and 3D Generalisations*
Many ConvNet ideas can applied to data other than 2D image data.
Can be applied to 1D data though more normally RNN's are used for sequential data.
For more dimensions you just increase the dimensions in your filter to match.
