---
layout: post
mathjax: true
title:  "Additional metrics other than the accuracy in classification"
date:   2019-12-02 13:33:00 +0200
categories: theory
author: Umberto Michelucci
github: https://github.com/michelucci
github-user: michelucci
---

I discuss briefly what other metrics are important when working on
classification problems, especially if dealing with unbalanced datasets.
The mathematics is discussed and especially I try to make the interpretation
clear and intuitive.
<!--more-->

# Additional metrics other than the accuracy in classification

Let's start briefly by discussing useful metrics that we can easily calculate
when dealing with classification.

- $tp$: true positives
- $fp$: false positives
- $tn$: true negatives
- $fn$: false negatives

Suppose we are doing some tests to determine if a subject has a certain disease or not.
We can interpret the quantities $tp$, $tn$, $fp$ and $fn$ as

- True positives ($tp$): Tests that predicted yes, and the subjects really have the disease
- True negatives ($tn$): Tests that predicted no, and the subjects do not have the disease
- False positives ($fp$): Tests that predicted yes, and the subjects do not have the disease
- False negatives ($fn$): Tests that predicted no, but the subjects do have the disease

This translates visually into the following (this is nothing else than a confusion
    matrix):

 **True vs. Prediction** | **Prediction: NO** | **Prediction: YES**
**True value: NO** | TRUE NEGATIVES | FALSE POSITIVES
**True value: YES** | FALSE NEGATIVES | TRUE POSITIVES

We can express several metrics as functions of the previously discussed terms. For
example (we indicate with $ty = tp + fn$ the number of patients who really have a disease, and
    with $tno = tn + fp$, the number of patients who don’t have a disease,)

- **Accuracy**: $(tp + tn)/N$, how often our test is right
- **Misclassification rate**: $(fp + fn)/N$, how often our test is wrong. Note that this is equal to $1 − accuracy$.
- **Sensitivity/Recall**: $tp/ty$, how often the test really predicts yes when the subjects have the disease
- **Specificity**: $tn/tno$, when the subjects have no disease, how often our test predicts no
- **Precision**: $tp/(tp + fp)$, the portion of tests predicting correctly the subject having the disease with respect to all positive results obtained

All those quantities can be used as metrics, depending on your problem. Let’s consider an example. Suppose your test should predict if a person has cancer or not. In this case, what you want is the highest sensitivity possible, because it is important to detect the disease. But at the same time, you also want the specificity to be high, because there is nothing worse than sending someone home without treatment when it is needed.
Let’s look a bit closer at **precision** and **recall**. Having a high precision means that when you say someone is sick, you are right. But you don’t know how many people really have the sickness, since the quantity is defined only by the result of your test. **Precision**
is a measure of how your test is doing. Having a high **recall** means that you can identify all the sick people in your sample. Let me give you another example, to make the point even clearer. Suppose we have 1000 people. Only 10 are sick and 990 are healthy. Let’s suppose we want to identify healthy people (this is important), and we build a test
that returns yes if someone is healthy and always predicts that people are healthy.

The confusion matrix would look like this:

**True vs. Predicted** | **Prediction: NO (Sick )** | **Prediction: YES (Healthy )**
**True: NO(Sick)** | 0 | 10
**True: YES (Healthy )** | 0 | 990

We would have

$$tp = 990$$

$$tn = 0$$

$$fp = 10 $$

$$fn = 0$$

That means that
- **Accuracy** would be 99%.
- The **misclassification rate** would be 10/1000 or, in other words, 1%.
- **Recall** would be 990/990 or 100%.
- **Specificity** would be 0%.
- **Precision** would be 99%.

This looks good, right? If want to find healthy people, this test would be great. The only problem is that it is a lot more important to identify sick people! Let’s recalculate the preceding quantities, but this time, considering that a positive result is when someone is sick. In this case, the confusion matrix would like this:

**True vs. Predicted** | **Prediction: NO (Healthy)** | **Prediction: YES (Sick)**
**True: NO(Healthy)** | 990 | 0%
**True: YES (Sick)** | 10 | 0%

because this time, a yes result means that someone is sick and not, as before, that
someone is healthy. Let’s calculate the quantities above again.

$$tp = 0$$

$$tn = 990$$

$$fp = 0$$

$$fn = 10$$


Therefore,

- **Accuracy** would still be 99%.
- The **misclassification rate** would still be 10/1000 or, in other words, 1%.
- **Recall** would now be 0/10 or 0%.
- **Specificity** would be 990/990 or 100%.
- **Precision** would be (0 + 0)/1000 or 0%.

Note how the accuracy remains the same. If you look only at that, you would not be able to understand how your model is doing. We have simply changed what we want to predict and use only accuracy. We cannot say anything about the performance of our model. But look at how recall and precision changed. See the matrix below for a comparison.

**Metric table** | **Predicting healthy people** | **Predicting sick people**
**Recall** | 100%  | 0
**Precision** | 99% | 0

Now we have something that changes that can give us enough information, depending on the question we pose. Note that changing what we want to predict will change how the confusion matrix will look. We can immediately say, looking at the preceding matrix, that our model that predicts that everyone is healthy works very well when predicting healthy people (not very useful) but fails miserably when trying to predict sick people.
There is another metric that is important to know, and that is the F1 score. It is defined as

$$
F1 = 2 \frac{precision \cdot recall}{precision + recall}
$$

An intuitive understanding is difficult to get, but it is basically the harmonic average of precision and recall. The example we created was a bit extreme and, having 0% for recall or precision, would not allow us to calculate F1. Let’s suppose that our model
is bad at predicting sick people, but not that bad. Let’s suppose we have the following confusion matrix:

**True vs. Predicted** | **Prediction: NO (Healthy)** | **Prediction: YES (Sick)**
**True: NO(Healthy)** | 985 | 5
**True: YES (Sick)** | 9 | 1

In this case, we would have (I leave the calculation to you)
- **Precision**: 54.5%
- **Recall**: 10%

We would have

$$
F1 = 0.169
$$

This quantity will give you information keeping precision (the portion of tests predicting correctly the subject having the disease with respect to all positive results obtained) and recall (how often the test really predicts yes when the subjects have the disease) in consideration. For some problems, you want to maximize precision, and for others, you want to maximize recall. If that is the case, simply choose the right metric.

Note that the F1 score will be the same for two cases in which you have **Precision**    = 32% and **Recall** = 45% and one with **Precision** = 45% and **Recall** = 32%. **Be aware of this fact**. Use F1 score, if you want to find a balance between Precision and Recall.

> Note the F1 score is used when you want to maximize the harmonic average of Precision and Recall, or, in other words, when you don’t want to maximize either Precision or Recall alone, but you want to find the best balance between the two.

If we calculate F1 when predicting healthy people, as we did at first, we would have

$$
F1 = 0.995
$$

This tells us that the model is quite good at predicting healthy people.
The F1 score is usually useful, because, normally, as a metric, you want one single number, and in this way, you don’t have to decide between precision or recall, as
both are useful. Remember that the values of the metrics discussed will always be dependent on the question you are asking (what is yes and no for you). Be aware that an interpretation is always dependent on the question you want to answer.

> Note remember that when calculating your metric, whatever it may be, changing your question will change the results. you must be very clear at the beginning of what you want to predict and then choose the right metric. in the case of highly unbalanced datasets, it is always a good idea to use not accuracy but other metrics as recall, precision, or, even better, F1, it being an average of precision and recall.
