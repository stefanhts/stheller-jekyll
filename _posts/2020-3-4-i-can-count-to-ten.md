---
layout: post
categories: posts
title: I can count to 10 in pictures
tags: [MachineLearning, code]
date-string: Mar 4, 2020
---
# Machine Learning is cooool!
I promised this in the last one, so here it is, a detailed explanation of one of my Machine Learning assignments. This is the first time we
have had a somewhat challenging, both computationally and intellectually, assignment and boy is it cool. Machine Learning has become a really
big thing over the last couple decades, and to contribute to its growth there have been many databases built which contain massive amounts of data
which you can use to test your networks and learn about ML. In this assignment we used a 60000 data point set which contained hand written digits from
0-9. The idea of the assignment was to create a network which could correctly classify the digit written.

![Digit array](/images/digit_ex.png)

This is what the input data look like in their raw form before we transform them into something more useful for our purposes.
<p/>
Before we can do anything of any significance, we have to import all the libraries that will carry us on our shoulders during this whirlwind of a journey.
```python
import tensorflow
from keras.models import Sequential
from keras.layers import Dense, Conv2D, Flatten, Dropout, MaxPooling2D
from keras.optimizers import Adam
from keras.datasets import mnist
from keras.utils import to_categorical, np_utils
import matplotlib.pyplot as plt
from keras.callbacks import History
import pandas as pd
```
We rely on a lot of dependencies here to make this project as streamlined as possible. Most of the tools we will use will come from Keras, a ML library which
greatly simplifies the work that goes into creating a Neural Net. This particular network is going to be what is called a 2D Convolutional Neural Network, which
essentially means that it uses convolution to identify patterns in two dimensional inputs to increase the effectiveness of the network. To make a CNN, we rely on classes
such as Conv2D, Flatten, etc. to provide us with key functionality, but we'll get to that in more detail later. Next we have to get all of our data and separate it into
training and testing inputs and outputs. This is very simple with the MNIST data, as it is built into Keras already. All we need to write is
```python
(tr_in, tr_out), (tst_in, tst_out) = mnist.load_data()
```
This is a simple and quick method to sort out our inputs and outputs, now we can move on to more interesting things.
<p/>
We start by normalizing and resizing our data in a way that our network can handle. Each pixel in our image will contain a value between 0-255, but we want this to be 0-1,
this is very straight forward and can be done simply in 2 lines:
```python
tr_in = tr_in / 255
tst_in = tst_in / 255
```
Our next step is to resize our data into a vector of a dimensionality that makes sense for 2-dimensional convolution, once again this is simple, although the values take a minute to
wrap your head around. <span style="color:orange;">tr_in.shape[0]</span> gives us the number of inputs and 28 is the width and height of each image.
```python
tr_in = tr_in.reshape(tr_in.shape[0], 28, 28, 1)
tr_in = tr_in.astype('float32')
tst_in = tst_in.reshape(tst_in.shape[0], 28, 28, 1)
tst_in = tst_in.astype('float32')
```
Next, we use One-Hot encoding to turn our training and testing output vectors into a size 10 output which contains the boolean value of which digit the image represents.
```python
num_classes = 10
tr_out = np_utils.to_categorical(tr_out, num_classes)
tst_out = np_utils.to_categorical(tst_out, num_classes)
```
where <span style="color:orange;">num_classes</span> is the total number of options between 0-9, thus 10.
<p/>
Now we come across the most difficult part of this entire quest, creating the model. This is a mainly trial and error process, and it's very possible you could build a model slightly
more accurate than mine, if so I congratulate you. My model consists of 4 2D convolutional layers, 2 pooling layers, and a dense layer.
```python
def create_model():
# create model
    model = Sequential()
    model.add(Conv2D(14, (3,3), input_shape=(28,28,1), activation = 'relu'))
    model.add(Conv2D(38, (3,3), activation = 'relu'))
    model.add(Dropout(0.3))
    model.add(MaxPooling2D((2,2)))
    model.add(Conv2D(28, (3,3), activation = 'selu'))
    model.add(Conv2D(52, (3,3), activation = 'selu'))
    model.add(Dropout(0.2))
    model.add(MaxPooling2D(2,2))
    model.add(Flatten())
    model.add(Dense(20, activation = 'selu'))
    model.add(Dense(10, activation = 'softmax'))
    # Compile model
    model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['acc'])
    return model
```
What we need to focus on here is our number of layers, types of layers, input shape, activations, loss function, optimizer, and metrics... so a lot. As for the layers, this is the
combination I found to work best for me. I use 4 convolutional filters to find patterns in the images, and then pool them twice. I throw in a couple dropouts to make sure my model isn't moving
in a direction which could slow or impede its progress. I flatten out the 2D data before finally puting it through a traditional dense layer and then normalize it once more for a 10 parameter output.
I find Adam to be the most effective optimizer for my purposes, and as for loss function and accuracy, categorical cross entropy and 'acc' are simple and effective functions for categorical outputs like we
have here.
<p/>
We then have to train model on our input data and evaluate it against our test data. We do this using the fit and evaluate functions build into Keras.
```python
model = create_model()
history = model.fit(tr_in, tr_out, batch_size = batch_size, epochs = epochs, validation_split = .1)
eval = model.evaluate(tst_in, tst_out)
```
![Output](/images/output.jpg)
When we run our code, this is the output we see. Over a measly 2 epochs, we see a testing accuracy of 97.2% which is a pretty reasonable accuracy for such a simplistic network. I find that if I run my program
multiple times, I find that my accuracy varies anywhere from 97-99%. There's a lot to be learned here, but overall I would call this a very successful first interesting project.

<p/>
My next post should be less technical than this, but if you found this interesting and would like me to do more similar to it, let me know.

See ya in the next one,
<br/>
Stefan
