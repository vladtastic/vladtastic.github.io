---
layout: post
title: "Thinking about Precision and Recall"
date: 2018-03-25
---

I got into a discussion today about precision and recall and how to describe them simply. We spent a good bit of time trying to explain these measures simply and I was trying to wrap my head around the differences between the two measures without confusing them. 

I think this is best explained with an example:

Let’s say I have ten cartons of milk lined up in a row on a table. I can’t read the label but I can observe a few things like the smell, the taste, etc. 

My goal is to use these things I can observe to guess which cartons are 2% milk. 

Let’s assume I’ve guessed 6 of the 10 cartons are 2% milk and in actuality 8 cartons are 2% milk. 

**Precision** is how many of the 6 milk cartons I guessed to be 2% actually are. 

**Recall** on the other hand is, given there are 8 cartons of 2% milk, how many did I find?

In some sense, these two are at odds because you have to make each individual guess correct but also have to figure out the right proportion of correct guesses. 

Here’s what I mean: In the milk carton example, if I guess every box of milk is 2%, I’ve identified all the 2% milk cartons that exist (*high recall*). Unfortunately, I’ve also made a lot of bad guesses since a lot of milk cartons that I thought were 2% milk actually were not (*low precision*). 

Or to put is simply: 

Precision is “Can you make the right individual guess?”

Recall is “How accurately can you find the truth?”

I think making the right individual guess might be easier. If you have 20 different pieces of information, all 20 may not be strong indicators. There may be 4 or 5 that give the strongest indication and the others may be noise or even misleading. 

Looking at correlations between variables seems like the best way to find the strongest indicators before training the model. 
