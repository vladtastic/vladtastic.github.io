---
layout: post
title: "Exploring the PIMS Indian Diabetes Database (Part 2)"
date: 2018-03-14
---

## [](#header-2) Recapping from Last Time
In the last post, I installed R, ggplot2 and did some initial data exploration of the PIMS Dataset. After looking through the data I concluded:

Some data cleaning is definitely necessary before splitting the data into train and test sets. In particular:

1 Entries with blood pressure of zero (there are 35) have to be handled since this isn't a realistic value...options are to drop them entirely, set their BP value to the mean or recategorize them somehow
2 Sample proportionally from diabetic and non-diabetic when creating the training and testing sets
3 It may be useful to transform some features into relative (to the mean) values (like number of pregnancies, glucose and insulin levels) since the absolute values mean less than relative measurements
4 Dropping the diabetes pedigree function feature since this doesn't seem to be intrisically an observed property like age or blood pressure

It occurred to me today that I hadn't asked two obvious questions: (1) _Is any of the data duplicated_? (2) Graphically, are relationships between the features?

Luckily, it's easy enough to check duplicates in R with the anyDuplicated function:

```R
> anyDuplicated(diabetes)
[1] 0
```

As for relationships between features, line plots weren't particularly helpful here but scatterplots did a better job, incl. highlighting some outliers.

![Blood Pressure v. Age]({{ "/assets/scatterBloodPressureAge.png" | absolute_url }} )

![Blood Pressure v. BMI]({{ "/assets/scatterBloodPressureBMI.png" | absolute_url }} )

![Blood Pressure v. Glucose]( {{ "/assets/scatterBloodPressureGlucose.png" | absolute_url }} )

![Blood Pressure v. Insulin]( {{ "/assets/scatterBloodPressureInsulin.png" | absolute_url }} )

## [](#header-2) Data Cleaning
Now let's revisit the 35 entries with Blood Pressure of zero. As it turns out, extracting these entries from the data frame reveals that this subset has zero values for other features like Insulin and SkinThickness:

```R
> noBP <- subset(diabetes, diabetes$BloodPressure == 0)

> noBP
    Pregnancies Glucose BloodPressure SkinThickness Insulin  BMI DiabetesPedigreeFunction Age
8            10     115             0             0       0 35.3                    0.134  29
16            7     100             0             0       0 30.0                    0.484  32
50            7     105             0             0       0  0.0                    0.305  24
61            2      84             0             0       0  0.0                    0.304  21
79            0     131             0             0       0 43.2                    0.270  26
82            2      74             0             0       0  0.0                    0.102  22
173           2      87             0            23       0 28.9                    0.773  25
194          11     135             0             0       0 52.3                    0.578  40
223           7     119             0             0       0 25.2                    0.209  37
262           3     141             0             0       0 30.0                    0.761  27
267           0     138             0             0       0 36.3                    0.933  25
270           2     146             0             0       0 27.5                    0.240  28
301           0     167             0             0       0 32.3                    0.839  30
333           1     180             0             0       0 43.3                    0.282  41
337           0     117             0             0       0 33.8                    0.932  44
348           3     116             0             0       0 23.5                    0.187  23
358          13     129             0            30       0 39.9                    0.569  44
427           0      94             0             0       0  0.0                    0.256  25
431           2      99             0             0       0 22.2                    0.108  23
436           0     141             0             0       0 42.4                    0.205  29
454           2     119             0             0       0 19.6                    0.832  72
469           8     120             0             0       0 30.0                    0.183  38
485           0     145             0             0       0 44.2                    0.630  31
495           3      80             0             0       0  0.0                    0.174  22
523           6     114             0             0       0  0.0                    0.189  26
534           6      91             0             0       0 29.8                    0.501  31
536           4     132             0             0       0 32.9                    0.302  23
590           0      73             0             0       0 21.1                    0.342  25
602           6      96             0             0       0 23.7                    0.190  28
605           4     183             0             0       0 28.4                    0.212  36
620           0     119             0             0       0 32.4                    0.141  24
644           4      90             0             0       0 28.0                    0.610  31
698           0      99             0             0       0 25.0                    0.253  22
704           2     129             0             0       0 38.5                    0.304  41
707          10     115             0             0       0  0.0                    0.261  30

Since this group seems to have a number of features missing that I think are important to predicting diabetes, and these records make up a small portion of the dataset (4%), I'll just drop them rather than investing time cleaning them. Also, at a glance it seems like these entries are random and not distinct from the rest of the dataset.

But having done this, are there entries with BP !=0 that have zero values features?

The answer is **yes** but the quantity depends on the feature...

```R
> nrow(subset(diabetes, diabetes$Age ==0))
[1] 0
> nrow(subset(diabetes, diabetes$Insulin ==0))
[1] 374
> nrow(subset(diabetes, diabetes$Glucose ==0))
[1] 5
> nrow(subset(diabetes, diabetes$SkinThickness ==0))
[1] 227
> nrow(subset(diabetes, diabetes$BMI ==0))
[1] 11
``` 

About half the dataset has Insulin as zero value and one-third for SkinThickness so I'll adjust all of these features (except Age) based on the mean.

```R
> cleanDiabetes <- subset(diabetes, diabetes$BloodPressure != 0)
> meanInsulin <- mean(cleanDiabetes$Insulin, na.rm=T)
> Insulin.fix <- ifelse(cleanDiabetes$Insulin==0, meanInsulin, cleanDiabetes$Insulin)
> nrow(subset(cleanDiabetes, cleanDiabetes$Insulin == 0))
[1] 339
> cleanDiabetes$Insulin <- Insulin.fix
> meanGlucose <- mean(cleanDiabetes$Glucose, na.rm=T)
> Glucose.fix <- ifelse(cleanDiabetes$Glucose==0, meanGlucose, cleanDiabetes$Glucose)
> cleanDiabetes$Glucose <- Glucose.fix
> meanSkinThickness <- mean(cleanDiabetes$SkinThickness, na.rm=T)
> meanBMI <- mean(cleanDiabetes$BMI, na.rm=T)
> SkinThickness.fix <- ifelse(cleanDiabetes$SkinThickness==0, meanSkinThickness, cleanDiabetes$SkinThickness)
> BMI.fix <- ifelse(cleanDiabetes$BMI==0, meanBMI, cleanDiabetes$BMI)
> cleanDiabetes$BMI <- BMI.fix
> cleanDiabetes$SkinThickness <- SkinThickness.fix
```

The cleaned data looks like this:

```R
> summary(cleanDiabetes)
  Pregnancies        Glucose    BloodPressure    SkinThickness      Insulin            BMI       
 Min.   : 0.000   Min.   :  0   Min.   : 24.00   Min.   : 0.00   Min.   :  0.00   Min.   : 0.00  
 1st Qu.: 1.000   1st Qu.: 99   1st Qu.: 64.00   1st Qu.: 0.00   1st Qu.:  0.00   1st Qu.:27.40  
 Median : 3.000   Median :117   Median : 72.00   Median :24.00   Median : 45.00   Median :32.30  
 Mean   : 3.855   Mean   :121   Mean   : 72.41   Mean   :21.44   Mean   : 83.61   Mean   :32.29  
 3rd Qu.: 6.000   3rd Qu.:141   3rd Qu.: 80.00   3rd Qu.:33.00   3rd Qu.:130.00   3rd Qu.:36.60  
 Max.   :17.000   Max.   :199   Max.   :122.00   Max.   :99.00   Max.   :846.00   Max.   :67.10  
 DiabetesPedigreeFunction      Age           Outcome      
 Min.   :0.0780           Min.   :21.00   Min.   :0.0000  
 1st Qu.:0.2450           1st Qu.:24.00   1st Qu.:0.0000  
 Median :0.3800           Median :29.00   Median :0.0000  
 Mean   :0.4759           Mean   :33.36   Mean   :0.3438  
 3rd Qu.:0.6290           3rd Qu.:41.00   3rd Qu.:1.0000  
 Max.   :2.4200           Max.   :81.00   Max.   :1.0000  
```

Wait, that's not right...Glucose, SkinThickness, Insulin, and BMI still have zero entries...It turns out the columns in the diabetes dataset were read in as vectors while the ones I've created were numeric vectors. Some type casting to vector as we're in business.

```R
> cleanDiabetes$Insulin <- as.vector(Insulin.fix)
> cleanDiabetes$Glucose <- as.vector(Glucose.fix)
> cleanDiabetes$SkinThickness <- as.vector(SkinThickness.fix)
> cleanDiabetes$BMI <- as.vector(BMI.fix)
> summary(cleanDiabetes)
  Pregnancies        Glucose      BloodPressure    SkinThickness      Insulin      
 Min.   : 0.000   Min.   : 44.0   Min.   : 24.00   Min.   : 7.00   Min.   : 14.00  
 1st Qu.: 1.000   1st Qu.:100.0   1st Qu.: 64.00   1st Qu.:21.44   1st Qu.: 83.61  
 Median : 3.000   Median :117.0   Median : 72.00   Median :24.00   Median : 83.61  
 Mean   : 3.855   Mean   :121.9   Mean   : 72.41   Mean   :27.12   Mean   :122.28  
 3rd Qu.: 6.000   3rd Qu.:141.0   3rd Qu.: 80.00   3rd Qu.:33.00   3rd Qu.:130.00  
 Max.   :17.000   Max.   :199.0   Max.   :122.00   Max.   :99.00   Max.   :846.00  
      BMI        DiabetesPedigreeFunction      Age           Outcome      
 Min.   :18.20   Min.   :0.0780           Min.   :21.00   Min.   :0.0000  
 1st Qu.:27.50   1st Qu.:0.2450           1st Qu.:24.00   1st Qu.:0.0000  
 Median :32.30   Median :0.3800           Median :29.00   Median :0.0000  
 Mean   :32.47   Mean   :0.4759           Mean   :33.36   Mean   :0.3438  
 3rd Qu.:36.60   3rd Qu.:0.6290           3rd Qu.:41.00   3rd Qu.:1.0000  
 Max.   :67.10   Max.   :2.4200           Max.   :81.00   Max.   :1.0000  
```
