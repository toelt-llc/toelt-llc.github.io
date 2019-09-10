---
layout: post
mathjax: true
title:  "TF2.0 Notes - Gradient Tape - quick overview"
date:   2019-09-09 21:34:54 +0200
categories: TensorFlow
---

This post contains some of my notes on ```GradientTape()``` API
from TensorFlow. The notes are kind of random but I hope they
are helpful with some useful examples.
<!--more-->

# A brief overview on how to use gradient tape in TF


First of all let's enable eager execution

    tf.enable_eager_execution()

The ```tf.GradientTape``` is there for automatic differentiation.
A dumb example

    x = tf.constant(1)

    with tf.GradientTape() as t:
      t.watch(x) # You need this code to let t knows that you need to
                 # keep track of x.
      y = 2*x

    dy_dx = t.gradient(y,x)

    print(dy_dx)

You will get

    tf.Tensor(2.0, shape=(), dtype=float32)

Since we have in the code

$$
y = 2x
$$

and therefore

$$
\frac{dy}{dx} = 2
$$

Note that TensorFlow operations executed within the context of a
```GradientTape``` are "recorded" so that the recorded computation can be later differentiated.  
So you should put the ```t.gradient()``` outside the context.

Note that according to the documentation "By default, the resources held by a GradientTape are
released as soon as ```GradientTape.gradient()``` method is called."
That means that you cannot call a second ```gradient()``` function with the code
above. Let's
suppose you want to evaluate two gradients, then you need the following code

    x = tf.constant(2.0)

    with tf.GradientTape(persistent = True) as t:
      t.watch(x) # You need this code to let t knows that you need to
                     # keep track of x.
      y = 2*x
      y1 = x*x*x

    dy_dx = t.gradient(y,x)
    dy1_dx = t.gradient(y1,x)

    print(dy_dx)
    print(dy1_dx)

Note the ```persistent = True``` as option whithin ```GradientTape```. In
this way you can do multiple calls with ```t.gradient()```.

## Example - MNIST

Fer example let's get the MNIST dataset and use the ```GradientTape``` to
develop our own training loop.

First things first. Let's load the dataset

    (x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()

    # Preprocess the data (these are Numpy arrays)
    x_train = x_train.reshape(60000, 784).astype('float32') / 255
    x_test = x_test.reshape(10000, 784).astype('float32') / 255

    y_train = y_train.astype('float32')
    y_test = y_test.astype('float32')

    # Reserve 10,000 samples for validation
    x_val = x_train[-10000:]
    y_val = y_train[-10000:]
    x_train = x_train[:-10000]
    y_train = y_train[:-10000]

We can use, for example, Keras functional APIs to define the model
we want to use

    inputs = keras.Input(shape=(784,), name='digits')
    x = layers.Dense(64, activation='relu', name='dense_1')(inputs)
    x = layers.Dense(64, activation='relu', name='dense_2')(x)
    outputs = layers.Dense(10, activation='softmax', name='predictions')(x)
    model = keras.Model(inputs=inputs, outputs=outputs)

    optimizer = keras.optimizers.SGD(learning_rate=1e-3)
    loss_fn = keras.losses.SparseCategoricalCrossentropy()

Then we can prepare the dataset (using ```tf.data.Dataset```)

    batch_size = 64
    train_dataset = tf.data.Dataset.from_tensor_slices((x_train, y_train))
    train_dataset = train_dataset.shuffle(buffer_size=1024).batch(batch_size)

And now we can do the actual loop

    for epoch in range(3):
      print('Start of epoch %d' % (epoch,))
      for step, (x_batch_train, y_batch_train) in enumerate(train_dataset):

        with tf.GradientTape() as tape:
          logits = model(x_batch_train)  # Logits for this minibatch
          loss_value = loss_fn(y_batch_train, logits)

        grads = tape.gradient(loss_value, model.trainable_weights)
        optimizer.apply_gradients(zip(grads, model.trainable_weights))

        if step % 200 == 0:
            print('Training loss (for one batch) at step %s: %s'
                 % (step, float(loss_value)))
            print('Seen so far: %s samples' % ((step + 1) * 64))

That should train pretty fast. Now the problem is that since
we have not compiled the model we cannot simply evaluate
the accuracy, or any metric we want. But is easy to solve.
We can simply add two lines during the model definition

    train_acc_metric = keras.metrics.SparseCategoricalAccuracy()
    val_acc_metric = keras.metrics.SparseCategoricalAccuracy()

then during the training (under the ```for``` loop over the batches)
we need

    train_acc_metric(y_batch_train, logits)

and then for each epoch we can print the metrics

    train_acc = train_acc_metric.result()
    print('Training acc over epoch: %s' % (float(train_acc),))
    train_acc_metric.reset_states()

and for example to do a loop over the ```val``` loops one could do

    for x_batch_val, y_batch_val in val_dataset:
        val_logits = model(x_batch_val)
        val_acc_metric(y_batch_val, val_logits)
      val_acc = val_acc_metric.result()
      val_acc_metric.reset_states()
      print('Validation acc: %s' % (float(val_acc),))
