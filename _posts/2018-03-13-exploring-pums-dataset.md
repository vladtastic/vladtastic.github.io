---
layout: post
title: "Exploring the PIMS Indian Diabetes Database (Part 1)"
date: 2018-03-13
---

## [](#header-2) Getting Started
I wanted to take a stab at creating a classification model, so I grabbed a copy of the [PIMS Indians Diabetes Dataset](https://www.kaggle.com/uciml/pima-indians-diabetes-database) and started exploring with RStudio. (I'm also making my way through Pratical Data Science with R, hence my choice of R as a data exploratory tool. Also from what I remember with R, it's easier than Python to do out of the box visualization).

The first hurdle I ran into was simply getting packages installed. I quickly realized I had a lot of packages and dependencies missing from system when I tried to load the ggplot2 library, Some of the dependencies were compiling from source and relying on other dependencies like libxml so things would attempt to compile and fail. A little bit of patience, installing some packages in Ubuntu using apt-get, and looking up some error messages in StackOverflow and I was able to get things sorted.

The next challenge was not crashing my computer. I loaded a few different datasets (of varying sizes) and when RStudio began to hang, I had my first clue that I might be running out of memory! Anyway, I decided to grab the PIMS Indian Diabetes dataset as it is small enough to easily run in memory and use to construct a model to predict diabetes. Given all the different features, the outcomes are either: has diabetes or not, so a logistic classifier would be a good starting point.

## [](#header-2) Exploring the Dataset
With the dataset loaded in R, we need to explore to data and understand what's in here. From the overview on Kaggle, we know:

* All the datapoints are from females
* Each person recorded is at least 21 years old

There are 8 features (not including the outcome):

1. number of pregnancies
2. glucose level
3. blood pressure
4. skin thickness
5. insulin level
6. body mass index (bmi)
7. diabetes pedigree function
8. age

The diabetes pedigree function isn't explained in the data overview but it may be explained in the cited paper. I'm guessing it's a measure of likelihood of diabetes.

Digging into the data itself, the first question I had is: "How many people were predicted to have diabetes vs not?". Turns out, roughly 1/3rd of the people were predicted to have diabetes. It's good to know this because it will affect how I create training and testing sets. (I could just guess that everyone is not diabetic (null model) and be very accurate on the training data)

```R
> diabetes <- read.table('diabetes.csv', sep=',', header=TRUE)
> summary(diabetes)
> summary(diabetes)
  Pregnancies        Glucose      BloodPressure    SkinThickness      Insulin     
 Min.   : 0.000   Min.   :  0.0   Min.   :  0.00   Min.   : 0.00   Min.   :  0.0  
 1st Qu.: 1.000   1st Qu.: 99.0   1st Qu.: 62.00   1st Qu.: 0.00   1st Qu.:  0.0  
 Median : 3.000   Median :117.0   Median : 72.00   Median :23.00   Median : 30.5  
 Mean   : 3.845   Mean   :120.9   Mean   : 69.11   Mean   :20.54   Mean   : 79.8  
 3rd Qu.: 6.000   3rd Qu.:140.2   3rd Qu.: 80.00   3rd Qu.:32.00   3rd Qu.:127.2  
 Max.   :17.000   Max.   :199.0   Max.   :122.00   Max.   :99.00   Max.   :846.0  
      BMI        DiabetesPedigreeFunction      Age           Outcome     
 Min.   : 0.00   Min.   :0.0780           Min.   :21.00   Min.   :0.000  
 1st Qu.:27.30   1st Qu.:0.2437           1st Qu.:24.00   1st Qu.:0.000  
 Median :32.00   Median :0.3725           Median :29.00   Median :0.000  
 Mean   :31.99   Mean   :0.4719           Mean   :33.24   Mean   :0.349  
 3rd Qu.:36.60   3rd Qu.:0.6262           3rd Qu.:41.00   3rd Qu.:1.000  
 Max.   :67.10   Max.   :2.4200           Max.   :81.00   Max.   :1.000  
> nrow(subset(diabetes, diabetes$Outcome == 0))
[1] 500
> nrow(subset(diabetes, diabetes$Outcome == 1))
[1] 268
```

|Diabetic  |Non-Diabetic |
|:---------|:------------|
|268       |500          |

![Histogram of Predicted Diabetics]({{ "assets/outcomeHistogramGG.png" | absolute_url }})

**Side note**: ggplot2 makes some _really_ nice plots. This one was generated using the following:
```R
ggplot2(diabetes) + geom_histogram(aes(x=diabetes$Outcome), binwidth=0.5)
```

Going through the other variables, other things stood out like:

* The highest number of pregnancies was **17**!
* Blood Pressure was 0 for _35 people_ (Maybe this did not get recorded or measured at the time) 
![Density of Blood Pressure]({{ "/assets/DensityBloodPressureGG.png" | absolute_url }})
* Average age was 33.24 but the max was **81** 
![Histogram of Ages]({{ "/assets/ageHistogramGG.png" | absolute_url }})
* Insulin levels reached a high of **846** while the average was 79.8
![Density of Insulin]({{ "/assets/densityInsulinGG.png" | absolute_url }})



## [](#header-2) Conclusions from Exploring the Data

Some data cleaning is definitely necessary before splitting the data into train and test sets. In particular:

* Entries with blood pressure of zero (there are 35) have to be handled since this isn't a realistic value...options are to drop them entirely, set their BP value to the mean or recategorize them somehow
* Sample proportionally from diabetic and non-diabetic when creating the training and testing sets
* It may be useful to transform some features into relative (to the mean) values (like number of pregnancies, glucose and insulin levels) since the absolute values mean less than relative measurements
* Dropping the diabetes pedigree function feature since this doesn't seem to be intrisically an observed property like age or blood pressure
