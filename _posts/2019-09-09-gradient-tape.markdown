---
layout: post
mathjax: true
title:  "TF Gradient Tape - quick overview"
date:   2019-09-09 21:34:54 +0200
categories: TensorFlow
---

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

Note that TensorFlow operations executed within the context of a
```GradientTape``` are "recorded" so that the recorded computation can be later differentiated.  So you should put the ```t.gradient()``` outside the context.

Note that according to the documentation "By default, the resources held by a GradientTape are released as soon as GradientTape.gradient() method is called."
That means that you cannot call a second ```gradient()``` function. Let's
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
