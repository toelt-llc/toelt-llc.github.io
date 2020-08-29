---
layout: post
mathjax: true
title:  "TF 2.0 Notes - Tensorboard in Jupyter - quick overview"
date:   2019-08-30 21:34:54 +0200
categories: visualisation
author: Umberto Michelucci
github: https://github.com/michelucci
github-user: michelucci
---

In this post I look quickly at how to use Tensorboard in
a jupyter notebook in TF 2.0.
<!--more-->

# A brief overview on how to use Tensorboard in a Jupyter notebook

I was checking how Tensorboard is integrated with TensorFlow 2.0, and so
I tought to write a few notes on how to use it. Not that difficult.

The tips I give you here works with Google Colab, or at least that
where I used them. They work also if you have a local installation of
Jupyter notebooks and Python (you may need to check a few things).

At the moment of writing the release candidate for TensorFlow 2.0 has been
released. To install it in Google Colab you can simply run

     !pip install tensorflow==2.0.0-rc0

in a cell in colab. Then you can simply start Tensorboard with a "magic" command

    %load_ext tensorboard

As you may know, TensorFlow will need to save information on the metrics and on
the model in a directory. Let's assume we will save everything in a ```log```
directory. In case you have old log files, or you want to clean everything and
start from scratch you can simply empty the directory with

     !rm -rf ./logs/

If you are interested in knowing more what magic commands are, you can check
[https://ipython.readthedocs.io/en/stable/interactive/magics.html](https://ipython.readthedocs.io/en/stable/interactive/magics.html). Lost to read there...

Now we can simply try something. We can simply get the MNIST dataset and create a model

    import tensorflow as tf
    import datetime, os
    fashion_mnist = tf.keras.datasets.fashion_mnist

    (x_train, y_train),(x_test, y_test) = fashion_mnist.load_data()
    x_train, x_test = x_train / 255.0, x_test / 255.0
    def create_model():
      return tf.keras.models.Sequential([
        tf.keras.layers.Flatten(input_shape=(28, 28)),
        tf.keras.layers.Dense(512, activation='relu'),
        tf.keras.layers.Dropout(0.2),
        tf.keras.layers.Dense(10, activation='softmax')
      ])
      model = create_model()
      model.compile(optimizer='adam',
                      loss='sparse_categorical_crossentropy',
                      metrics=['accuracy'])

now comes the interesting part. We need to save all files that tensorboard
needs in a directory. For each run you want to create a specific directory.
One way is to simply give it a name

    logdir = "logdir-run1"

for example. In case you want to do something a bit fancier, you could
create a directory that contains the date and time. This piece of code
can be found also on the tensorflow website

    logdir = os.path.join("logs", datetime.datetime.now().strftime("%Y%m%d-%H%M%S"))

The downsize of this approach is that you will get soon quite few directories
and you will have no ideas of what is in which. So if you are doing some
controlled runs, think about some naming conventions. Tensorboard by default
will visualise every single run that will find in the ```logs``` directory, and
the plot will become very full very soon. You are warned.

If you are using Keras, you will need to use a ```callback``` class to write
the log information with a specific frequency. For example

    tensorboard_callback = tf.keras.callbacks.TensorBoard(logdir, histogram_freq=1)

The class ```tf.keras.callbacks.TensorBoard``` inherits from
```tf.keras.callbacks.Callback```. That means that in principle you can customize
it. But that is something for another post. The last point you need to
pass it to the ```fit``` method with the ```callbacks``` option, as in

    model.fit(x=x_train,
              y=y_train,
              epochs=15,
              validation_data=(x_test, y_test),
              callbacks=[tensorboard_callback])

At this point, in the tensorboard window you should start to see the plots
of the metric chosen (in this case the accuracy, as we have defined in the
```compile```).

In my opinion is a good solution, but not yet ideal. But is nice to be able
to see the diagram of the network and how the training is going. This can be quite useful.
