# TensorFlow: Simple MNIST example with softmax

## Overview

We'll go throught the example of analysing some hand-written numbers and classifying them. 
See http://www.tensorflow.org/tutorials/mnist/beginners/index.html

## Getting the MNIST data

It's locate at http://yann.lecun.com/exdb/mnist/ but there is a python script that downlaods it directly
into python. Wgetting didn't work, but copy&pasting it did. Here is the [file.](https://tensorflow.googlesource.com/tensorflow/+/master/tensorflow/g3doc/tutorials/mnist/input_data.py)

Activate tensorflow (source ~/tensorflow/bin/activate) and in python enter:
```
>>> import input_data
>>> mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting MNIST_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting MNIST_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting MNIST_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting MNIST_data/t10k-labels-idx1-ubyte.gz
```
## First test: SoftMax Regression Model
Once downloaded we can proceed with the first ML test. It uses  softmax with gradient descent:
```
>>> x = tf.placeholder(tf.float32,[None,784])
>>> W = tf.Variable(tf.zeros([784, 10]))
>>> b = tf.Variable(tf.zeros([10]))
>>> y = tf.nn.softmax(tf.matmul(x,W)+b)
>>> y_ = tf.placeholder(tf.float32, [None, 10])
>>> cross_entropy = -tf.reduce_sum(y_*tf.log(y))
>>> train_step = tf.train.GradientDescentOptimizer(0.01).minimize(cross_entropy)
>>> init = tf.initialize_all_variables()
>>> sess = tf.Session()
>>> sess.run(init)
>>> for i in range(1000):
...   batch_xs, batch_ys = mnist.train.next_batch(100)
...   sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})
...
>>> correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
>>> accuracy = tf.reduce_mean(tf.cast(correct_prediction, "float"))
>>> print sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels})
0.905
>>> sess.close()
```

## Deep Neural Network: Multilayer Convolutional Network
```
>>> sess = tf.InteractiveSession()
>>> def weight_variable(shape):
...   initial = tf.truncated_normal(shape, stddev=0.1)
...   return tf.Variable(initial)
...
>>> def bias_variable(shape):
...   initial = tf.constant(0.1, shape=shape)
...   return tf.Variable(initial)
...
>>> def conv2d(x, W):
...   return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')
...
>>> def max_pool_2x2(x):
...   return tf.nn.max_pool(x, ksize=[1, 2, 2, 1],
...                         strides=[1, 2, 2, 1], padding='SAME')
...
>>> W_conv1 = weight_variable([5, 5, 1, 32])
>>> b_conv1 = bias_variable([32])
>>> x_image = tf.reshape(x, [-1,28,28,1])
>>> h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
>>> h_pool1 = max_pool_2x2(h_conv1)
>>> W_conv2 = weight_variable([5, 5, 32, 64])
>>> b_conv2 = bias_variable([64])
>>>
>>> h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
>>> h_pool2 = max_pool_2x2(h_conv2)
>>> W_fc1 = weight_variable([7 * 7 * 64, 1024])
>>> b_fc1 = bias_variable([1024])
>>>
>>> h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
>>> h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)
>>> keep_prob = tf.placeholder("float")
>>> h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)
>>> W_fc2 = weight_variable([1024, 10])
>>> b_fc2 = bias_variable([10])
>>>
>>> y_conv=tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)
>>> #comment
...
>>> #Train and Evaluation
...
>>> cross_entropy = -tf.reduce_sum(y_*tf.log(y_conv))
>>> train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
>>> correct_prediction = tf.equal(tf.argmax(y_conv,1), tf.argmax(y_,1))
>>> accuracy = tf.reduce_mean(tf.cast(correct_prediction, "float"))
>>> sess.run(tf.initialize_all_variables())
>>> for i in range(20000):
...   batch = mnist.train.next_batch(50)
...   if i%100 == 0:
...     train_accuracy = accuracy.eval(feed_dict={
...         x:batch[0], y_: batch[1], keep_prob: 1.0})
...     print "step %d, training accuracy %g"%(i, train_accuracy)
...   train_step.run(feed_dict={x: batch[0], y_: batch[1], keep_prob: 0.5})
...
step 0, training accuracy 0.02
step 100, training accuracy 0.9
step 200, training accuracy 0.92
step 300, training accuracy 0.9
step 400, training accuracy 0.94 (time: 7:40)
step 500, training accuracy 0.92 (time: ~9:00)
step 600, training accuracy 1	 (time: 11:14)
step 700, training accuracy 0.92 (time: 12:59)
step 800, training accuracy 0.92
step 900, training accuracy 0.96
step 1000, training accuracy 1
step 1100, training accuracy 0.92
step 1200, training accuracy 0.98
step 1300, training accuracy 1	 (time: ~24:30)
step 1400, training accuracy 0.96
step 1500, training accuracy 0.98
step 1600, training accuracy 0.94
step 1700, training accuracy 1
step 1800, training accuracy 0.96
step 1900, training accuracy 0.98
step 2000, training accuracy 0.96 (time: 36:51)
step 2100, training accuracy 0.96 (time: 38:45)
step 2200, training accuracy 1
step 2300, training accuracy 0.98
step 2400, training accuracy 1
step 2500, training accuracy 1
step 2600, training accuracy 1
step 2700, training accuracy 1
step 2800, training accuracy 1
step 2900, training accuracy 1
step 3000, training accuracy 1
step 3100, training accuracy 0.94
step 3200, training accuracy 1
step 3300, training accuracy 0.98
step 3400, training accuracy 0.98
step 3500, training accuracy 1
step 3600, training accuracy 0.98
step 3700, training accuracy 1	 (time: 68:12)
step 3800, training accuracy 0.98
step 3900, training accuracy 1
step 4000, training accuracy 0.96
step 4100, training accuracy 0.98
step 4200, training accuracy 1
step 4300, training accuracy 0.94
step 4400, training accuracy 1
step 4500, training accuracy 1
step 4600, training accuracy 1
step 4700, training accuracy 0.98
step 4800, training accuracy 1
step 4900, training accuracy 0.98
step 5000, training accuracy 0.98
step 5100, training accuracy 1
step 5200, training accuracy 1
step 5300, training accuracy 1
step 5400, training accuracy 1
step 5500, training accuracy 1
step 5600, training accuracy 1
step 5700, training accuracy 1
step 5800, training accuracy 1
step 5900, training accuracy 0.96
step 6000, training accuracy 1
step 6100, training accuracy 1
step 6200, training accuracy 1
step 6300, training accuracy 1
...
...
step 10000, training accuracy 0.98
step 10100, training accuracy 1
step 10200, training accuracy 1
step 10300, training accuracy 1
step 10400, training accuracy 1
step 10500, training accuracy 0.98
step 10600, training accuracy 1
step 10700, training accuracy 1
step 10800, training accuracy 1
step 10900, training accuracy 1
step 11000, training accuracy 1
step 11100, training accuracy 1
step 11200, training accuracy 1
step 11300, training accuracy 1
step 11400, training accuracy 1
step 11500, training accuracy 1	 (time: 217:01)

```
After 5min the accuracy is 92% and after 6min is back down to 90%. After around 25 it approaches 1 steadily.
The training has been set to 20k iterations.

It took about 217min for 11500 iterations, which is about 1.9min for 100iterations and gives an ETA time of 
around 160min, with an expected total computation time of ~6hours using 1single core of my MacBook Pro early 2009.

The memory footprint is not significant as top shows here
```
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 8494 msantos   20   0  742568 573864  33928 S 99.3 28.0 239:50.68 python
````
The last steps
```
step 18800, training accuracy 0.98
step 18900, training accuracy 1
step 19000, training accuracy 1
step 19100, training accuracy 1
step 19200, training accuracy 1
step 19300, training accuracy 1
step 19400, training accuracy 1
step 19500, training accuracy 1
step 19600, training accuracy 1
step 19700, training accuracy 1
step 19800, training accuracy 1
step 19900, training accuracy 1
```
Notice the code doesn't print the accuracy of step 20000, i.e., the last step.

More soon...

