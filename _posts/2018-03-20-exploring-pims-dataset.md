---
layout: post
title: "Exploring the PIMS Indian Diabetes Database (Part 3)"
date: 2018-03-20
---

## [](#header-2) Model Performance

I was able to run the model last time but didn't get a chance to post the results. Naturally the question is, how did my model DO?

I initially set the threshold of the model to be ε=0.375 and measured precision, recall and enrichment:

|threshold (ε) |0.375|
|--------------|-----|
|precision     |-~54%|
|recall        |-~57%|
|enrichment    |-1.75|

where each measure can be thought of as:
* precision: how many of the predicted positive classifications were correct?  (correct pos/all pos guesses)
* recall: how many true positive items were discovered? (correct pos/ [correct pos + incorrect neg] )
* enrichment: ratio of precision to average rate of positives

So not great performance the first time around but it does raise some questions that may help tune the model, like what about adjusting the threshold of the model, and how can we even evaluate the performance of the model?

The first is pretty easy to find out since the threshold is easily adjustable...

|Threshold|Precision|Recall|Enrichment|F1  |
|---------|:-------:|:----:|---------:|
|0.2      |0.46     |0.77  |1.377     |
|0.25     |0.46     |0.775 |1.480     |
|0.375    |0.5476   |0.575 |1.7523    |
|0.4	  |0.575    |0.575 |1.84      |
|0.5      |0.6363   |0.525 |2.03636   |
|0.6      |0.6667   |0.45  |2.1333    |


From this, and the plot of the distribution of positive and negative predictions, the two distributions are separated (which led me to pick a threshold of ~37.5% initially). Since the distributions are separated, the model can somewhat balance precision and recall. In fact, increasing the threshold to 0.4 gives the best balance of precision and recall.

As for performance measurements, there are a few things to measure, each giving different insight into the model performance:
* precision
* recall
* enrichment
* ROC curve
* number of Fisher iterations
* chi-squared test
* pseudo R-squared
* Akaike information criterion

## [](#header-2) Potential Improvements

There are a few things which may improve the performance (potential tweaks for next time)

1 Scaling absolute values by the mean
1.1 Although I replaced zero values by the mean, the nonzero values could still possibly and in some cases do have a very wide range. It would be good to try scaling many features by their mean
2  Looking at correlation of variables to try and identify relationships in the data (and reduce the number of features)
3 k-fold cross validation of the model

