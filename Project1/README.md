
---
layout: page
mathjax: true
title: Image Classification Using Convolutional Neural Networks
permalink: /2019/proj/p4/
---

Table of Contents:
- [Deadline](#due)
- [Introduction](#intro)
- [Implementation Overview](#system_overview)
- [Submission Guidelines](#sub)
- [Collaboration Policy](#coll)

<a name='due'></a>
## Deadline 
11:59 PM, May 17, 2019

<a name='intro'></a>
## Introduction
In this project you will implement a convolutional neural network (CNN). You will be building a supervised classifier to classify MNIST digits dataset.  

Supervised classification is a computer vision task of categorizing unlabeled images to different categories or classes. This follows the training using labeled images of the same categories. You will be provided with a data set of images of MNIST digits. All of these images will be specifically labeled as a specific digit. You would use these labeled images as training data set to train a convolutional neural network. Once the CNN is trained you would test an unlabeled image and classify it as one of the ten digits. This task can be visualized in Figure 1.

<div class="fig fighighlight">
  <img src="/assets/proj4/mnist.png" width="100%">
  <div class="figcaption">
  </div>
  <div style="clear:both;"></div>
</div>

## Architecture
Your training and test step would contain the pipeline shown in Figure 2

<div class="fig fighighlight">
  <img src="/assets/proj4/architecture.png" width="100%">
  <div class="figcaption">
  </div>
  <div style="clear:both;"></div>
</div>
You are provided a framework for a single convolution layer (convolution + subsampling), a fully connected neural network with Sigmoid activation function, and a cross-entropy softmax layer with ten outputs.  Next few sections present a description of each of these components of this network. 

## Fully Connected Layer
In a fully connected layer each neuron is connected to every neuron of the previous layer as shown in Figure 3
<div class="fig fighighlight">
  <img src="/assets/proj4/fullyconnected.jpg" width="40%">
  <div class="figcaption">
  </div>
  <div style="clear:both;"></div>
</div>

Each of these arrows (inputs) between the layers is associated with a weight. All these input weights can be represent by a 2-dimensional weight matrix, <b>W</b>. If the number of neurons in this layer is n<sub>c</sub> and the number of neurons in the previous layer is n<sub>p</sub>, then the dimensionality of the weight matrix <b>W</b> will be n<sub>c</sub> x n<sub>p</sub>. There is also a bias associated with each layer. For the current layer we will represent it by a vector, <b>b</b>,  with a size of n<sub>c</sub> x 1. Therefore, for a given input, <b>X</b>, the output, <b>Z</b>, of a fully connected layer is given by the equation:

Z = WX + b

## Convolutional layer
We will begin with a brief description of cross-correlation and convolution which are fundamental to a convolutional layer. Cross-correlation is a similarity measure between two signals when one has a time lag, represented in the continuous domain as

<img src="/assets/proj4/crossCorrEq.jpg" width="40%">

Convolution is similar, although one signal is reversed

<img src="/assets/proj4/conv.jpg" width="25%">

They have two key  features, shift invariance and linearity. Shift invariance means that the same operation is performed at every point in the image and linearity means that every pixel is replaced with a linear combination of its neighbors.

In a convolutional layer an image is convolved with a filter. For example a 3 x 3 filter can be represented as

<img src="/assets/proj4/filter.jpg" width="25%">

Each number in this filter is referred to as a weight. The weights provided in this filter are example weights. Our goal is to learn the exact weights from the data during convolution. For each convolution layer a set of filters is learned. Use of convolution layer has many features but two stand out:
<ul>
<li> Reduction in parameters<br>
  In a fully connected neural network the number of parameters is much larger than a convolution network. Consider an image of size 5 x 5 to be convolved with a filter of size, 3 x 3. The number of weights we would have to learn would be the total number of weights in the filter, which is, 9. On the contrary for the same image, using a fully connected layer would require us to learn 225 weights. This is demonstrated in below Figure
  
  <img src="/assets/proj4/params.jpg" width="100%">
  </li>
  <li> Exploitation of spatial structure <br>
  Our images are in two-dimensions. In order to use them as an input to a fully connected layer, we need to unroll it into a vector and feed it as an input. This leads to the increase in the number of parameters to be learned as shown in the previous section and loss of the original 2D spatial structure. On the contrary, in convolution layer we can directly convolve a 2D image with a 2D filter thereby preserving the original spatial structure and also reducing the number of parameters to be learned.
  </li>
  </ul>

<a name='system_overview'></a>
## Implementation Overview
Convolution involves multiplying each pixel in an image by the filter at each position, then summing them up and adding a
bias. The figure below shows a single step of convolving an image with a filter. Each convolution will return a 2D matrix output for each input channel.

<img src="/assets/proj4/convolution.jpg" width="60%">

For example, if the input matrix is of size nxn and the filter is of size fxf, then the convolution output matrix will be of the size (n-f+1)x(n-f+1). The convolution output size remains the same even when there are more than one channels. For example, if there is an RGB image of size (nxnx3) and a filter of size (3x3x3), both with three channels, the convolution output will still be (n-f+1)x(n-f+1). However, if we convolve an image with more than a single filter then we will get a convolved image for each of those filters. For example, an image of size, (nxnx3), convolved with two filters, each of size, (nxnx3), will result in an output of size, (n-f+1)x(n-f+1)x2.

## Padding 
You probably noticed that with convolution the image gets shrunk in size. This would be a problem in a large network. However, we can retain the size of convolution output by zero-padding the image as shown in the figure below

<img src="/assets/proj4/padding.jpg" width="75%">

Padding helps to build deeper networks and keep more information at the border of an image. If there is an image of size nxnx1 and it is convolved with a filter of size fxfx1, in order to retain the original size, the output image would have to be of the size (n+2p-f+1) x (n+2p-f+1), where p is the zero-padding size, given by:

<img src="/assets/proj4/paddingeq.jpg" width="10%">

## Strides
The number of pixels we slide between each step of the convolution of an image and a filter is called a stride. So far we have been sliding one pixel at a time and therefore the stride is 1. However, we could skip a pixel and the stride would be 2, if we skip 2 pixels the stride value will be 3, and so on and so forth. Depending on the value of the stride, the output image has the size,

<img src="/assets/proj4/strideeq.jpg" width="30%">

where the input image size is n x n, zero-padded with p columns and rows, with a stride, s, and convolved with a filter of size, f x f.

## Pooling layer
The pooling layer shrinks the size of the input image. Reduction in size reduces the computation. Max-pooling layer is demostrated by an example in the below figure

<img src="/assets/proj4/maxpooling-2.jpg" width="60%">

Maxpooling is similar to convolution, except, instead of convolving with a filter, we get the max value in each kernel. In this example, we use a max of f x f kernels of the image, with a padding of 0 and a stride of 1.

Similarly, average pooling takes the average of each kernel as shown in teh figure below:

<img src="/assets/proj4/averagepooling.png" width="40%">

## Activation layer

In order to learn complex and non-linear features we often need a non-linear function. One of the most common non-linear functions is a rectified linear unit (ReLU), as shown in Figure below

<img src="/assets/proj4/ReLU.png" width="40%">

This function is applied to each output of the previous layer and is defined as,

<img src="/assets/proj4/relueq.jpg" width="20%">

and demonstrated in the following figure

<img src="/assets/proj4/relu-conv.jpg" width="70%">

### Code

Some of the code is already implemented for you. The details of the starter files that are provided to you are as follows:<br>
<ul>
  <li> cnnTrain.m<br>
    This is the driver file. The data sets are loaded from this file and so are various other functions
  </li>
  <li>cnnCost.m <br>
    This file contains the code for backpropagation. It is already implemented for you.
  </li>
  <li>config.m <br>
    
    It is a configuration file where the network components are specified.
</li>
<li> All the test code is in the test directory</li>
</ul>
All of these and other starter code files can be accessed from the following link: <a href="https://drive.google.com/drive/folders/1bIbK2fin-6Qnz0Jb_BCxNci1u4kb1umc">https://drive.google.com/drive/folders/1bIbK2fin-6Qnz0Jb_BCxNci1u4kb1umc</a>

## What to Implement

You're supposed to implement the following:
<ul>
  <li> Forward Pass<br>
    Forward pass algorithm to train your Convolution Neural Network. This will work in conjunction with the backpropagation algorithm  that is provided to you. You will be implementing this algorithm in <b>cnnConvolve.m</b> file.
  </li>
  <li> Subsampling<br>
    After convolution layer, subsample the output by implementing the following two layers in cnnPool.m :
   <ul>
     <li>maxpool</li>
     <li>averagepool</li>
   </ul>   
 </li>
 <li> Configuration<br>
    Although you wont't need to change either the fully connected layer or the softmax layer
you would require more than one convolution and the subsampling layer. In order to add new layers you would have to modify config.m file. The new layers would have to go after the following lines in the file:<br>
   cnnConfig.layer{2}.type = 'conv';<br>
cnnConfig.layer{2}.filterDim = [9 9];<br>
cnnConfig.layer{2}.numFilters = 20;<br>
cnnConfig.layer{2}.nonLinearType = 'sigmoid';<br>
cnnConfig.layer{2}.conMatrix = ones(1,20);<br>
cnnConfig.layer{3}.type = 'pool';<br>
cnnConfig.layer{3}.poolDim = [2 2];<br>
cnnConfig.layer{3}.poolType = 'maxpool';

<br>
<br>
However, make sure you change the index of the subsequent layers since the numbers are sequential.
  </li>
 </ul>
    

<a name='sub'></a>
## Submission Guidelines
<b> We will deduct points if your submission does not comply with the following guidelines.</b><br>
You're supposed to submit the following:<br>
<ul>
  <li> All the files that contain your code: cnnConvolve.m, cnnPool.m, config.m or any other files where you implement your code or make modifications.</li>
  <li> Accuracy for a single convolution and subsampling layer with Sigmoid activation function.</li>
  <li> Accuracy for a single convolution and subsampling layer with a ReLu activation function.</li>
  <li> Compare the results between maxpooling and average pooling.</li>
  <li> Try with more than one convolution and subsampling layers for both sigmoid and ReLu activation functions. </li>
  <li> Submit a confusion matrix and the accuracy for the best configuration. </li>
  <li> Dimensions of the Input and output of each layer (convolution(s), maxpool(s) / average pool(s), fully connected layer, and softmax layer) for your best config.</li>
</ul>
<br>
Please submit the project <b> once </b> for your group -- there's no need for each member to submit it.



<a name='coll'></a>
## Collaboration Policy
We encourage you to work closely with your groupmates, including collaborating on writing code.  With students outside your group, you may discuss methods and ideas but may not share code.  

For the full collaboration policy, including guidelines on citations and limitations on using online resources, see <a href="https://cmsc426spring2019.github.io/index.html">the course website</a>.
