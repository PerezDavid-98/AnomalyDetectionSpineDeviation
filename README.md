# Anomaly Detection for a Spine Deviation dataset

For this project I did an Anomaly Detection Study using the `R` programming language for a commonly used dataset in the literature, tailored for Anomaly Detection tasks. 

The whole study, including implementations, can be read (in Spanish) as an `HTML` file in this repo or in the next link: [AnomalyDetectionSpineDeviation](https://perezdavid-98.github.io/AnomalyDetectionSpineDeviation/AnomalyDetectionSpineDeviation.html).

## Dataset
The dataset used is the Vertebral Column Dataset from the UCI Machine Learning Repository, which has been modified for outlier detection, obtained from: [ODDS](http://odds.cs.stonybrook.edu/vertebral-dataset/).

> [!WARNING] 
The ODDS (Outlier Detection DataSets) page cannot be reached as of the moment of writing this README. It returns an `SSL_ERROR_NO_CYPHER_OVERLAP` error. 
I will link a 3rd party information page: ([paperswithcode](https://paperswithcode.com/dataset/odds)) about the ODDS page and delete this warning if the ODDS page is up again.

This dataset has 240 instances where the following convention is used for the classes: 

- Normal (NO)
- Abnormal (AB)

Which refer to the deviation of the patient's spine. `AB` is the majority class, with 210 instances used as inliers, and `NO` is reduced from 100 to 30 instances as outliers. The dataset downloaded from the ODDS page was in .mat format (Matlab file); for convenience, it was converted to CSV before reading it in `R`.

## Implementation Overview

Everything is implemented in an `R Notebook` file `.Rmd`. I will not go over everything in the README of the repo, as it can be found in the html or in the notebook itself, but I will go over some implementation decisions and some important data objects.

### Dataset objects
To work with the dataset, the following objects were created:

- `dataset`: Data frame that will contain the `vertebral` dataset.

- `dataset.num`: Frame obtained from `data` using only numeric columns. (Fr this dataset, all columns are numeric so this object will be equal to the `dataset` object)

- `index.column`: Index of the `data` column that will be worked with.

- `column`: Contains the `dataset` column corresponding to `index.column`.

- `name.column`: Name of the column corresponding to `index.column`.

### Outliers in 1 Dimension

In this section, the 1-variant outliers (or _Univariate Outliers_) are calculated.

The IQR method was applied to check for Univariate Outliers. First a check was done to know whether the variables followed a normal distribution, as the IQR method cannot be used otherwise.

I decided to use the `lumbar_lordosis_angle` variable since it closely resembles a normal distribution. (based on histogram representation)

After the IQR method was applied, it could be seen how the patient `no. 198` had an excessive value in the `lumbar_lordosis_angle` variable. 
The only other column where it showed a somewhat elevated value was `degree_spondylolisthesis`. This may make sense since pelvic morphology may play a role in the development of [spondylolisthesis](https://www.nhs.uk/conditions/spondylolisthesis/).

### Hypothesis testing

A hypothesis test was used to determine whether the value furthest from the mean could be considered an outlier. To do this, the `Grubbs` test was used, where the following null hypothesis was proposed:

> H0: The value furthest from the mean comes from the same distribution as the rest of the data.

To be able to use the `Grubbs` test, it must be taken into account that it assumes that the data follows a normal distribution without taking into account the outlier, so the normality of the data must be checked if the null hypothesis of the test is rejected in order to affirm, with statistical certainty, that the value is an outlier.

To check the normality of the distribution, the following null hypothesis is established:

> H0: The underlying distribution of the variable is a Normal distribution

Depending on the number of instances, several tests can be used. Since the dataset has 240 examples, the `Shapiro-Willks` test could not be used, so the `Kolomogorov-Smirnov` test was used instead, specifically the `Lilliefors` version was used.

Other tests like the `Anderson-Darling` test could not be applied due to limited data. The `Kolomogorov-Smirnov` test could not reject the null hypothesis of Normality (p-value > 0.05). Therefore, it could be assumed that the data did not contradict the assumption that the underlying distribution is, in fact, a Normal distribution.

In summary, it could be concluded that the distribution of `lumbar_lordosis_angle` is compatible with a Normal distribution and that patient 198, with a value of 125.74, is the one that is furthest from the mean and can be considered a statistically guaranteed outlier according to the `Grubbs` test.


This process was then repeated fr the rest of the variables (Graphical representation, Hypothesis tests and IQR method application). 

### Multivariate Outliers

In this section, the objective was to find multivariate outliers, using statistical techniques, distance-based techniques, and clustering-based techniques.

#### Statistical methods based on the Mahalanobis distance

To find multivariate outliers using statistical techniques, techniques based on the Mahalanobis distance were used. The purpose of this was to provide statistical assurance that if a value is labeled as an outlier, it truly is one. Therefore, the null hypothesis states that it is not an outlier, so if it is rejected, it is certain that it is an outlier:

> H0: The value furthest from the center of the distribution is not an outlier.

Methods based on the Mahalanobis distance assume that the joint distribution is a multivariate Normal distribution. Therefore, the null hypothesis is actually the following:

> H0: The value with the greatest Mahalanobis distance from the center of the distribution comes from the same multivariate Normal distribution as the rest of the data.

The test results indicate that the joint distribution of the variables `lumbar_lordosis_angle` and `sacral_solpe` is not a multivariate Normal distribution. Therefore, the Mahalanobis distance method should not be applied. In any case, the test was run to see if any values were detected that, while not statistically guaranteed to be outliers, at least provide some interesting information.

#### Distance-based methods: LOF
Statistical methods aim to provide statistical assurance that values labeled as outliers actually are outliers. To do so, they assume that the data follows a specific statistical distribution. However, in situations where this requirement is not met, these methods cannot be applied. In these cases, other methods were applied that do not offer statistical assurance but are capable of determining how far each point is from the rest of the data. To do this, an Euclidean distance measure is used. To ensure that some variables do not dominate over others, the data must be normalized. Z-score normalization was be used. Out of the distance-based methods, `LOF` will be applied.

#### Clustering-based methods

For the Clustering based methods, a k-means model is used. 

In the absence of further information, the number of outliers is set to 5 and the number of clusters to 2. Furthermore, since the results of the k-means clustering method depend on the initial choice of centroids, a seed value is set using the `set.seed` function so that the results do not vary from one run to the next.

The outliers can be calculated as those data points that are furthest from the centroid of the cluster to which they have been assigned.

The PAM (Partition around medoids) clustering method is also used. The distance matrix from all to all must first be calculated using the `dist` function in `R`.


### Analysis of pure multivariate outliers

Pure multivariate outliers are also analyzed. To do this, single-variable outliers will be identified using the IQR method.

It will therefore be sufficient to examine the outliers that are multivariate, but not univariate (with respect to any variable). This process can be applied to any of the methods discussed previously (statistical, distance-based, or clustering-based). The LOF method was used, which is one of the methods that yields the best results in a wide variety of situations.


## Overall Conclusions 

Throughout the study, several outliers were found using different approaches.

A summary of the Outliers found based on the method used to find it can be found below:

- IQR Method

    - Extreme outliers (more than three times the interquartile range from the mean) and non-extreme outliers (more than 1.5 times the interquartile range from the mean) were found.

    - Patient `198` spikes in `lumbar_lordosis_angle` but not as much in the other columns. Therefore, they appear to be a fairly balanced patient with a pronounced lower back curvature. Furthermore, this outlier can be considered a real outlier with statistical guarantee.

    - The case of Patient `116` is interesting, as they have spikes in several columns: `pelvic_incidence`, `sacral_slope` and `degree_spondylolisthesis`. These values show that the patient has severe lower back pain, which makes sense since pelvic incidence and sacral slope are very similar measurements located in the lower back, and spondylolisthesis is more common in the lower back.

- LOF Method

    - The score graph showed one notable case: patient `116`. This patient obtained a high score due to extreme values in several variables. The relationship between the variables with extreme values is as explained above: pelvic incidence and sacral slope are very similar measurements located at the junction between the spine and pelvis, and spondylolisthesis is more common in the lower back. Even so, it cannot be statistically concluded that this is a pure multivariate outlier. Patient no. `116` stands out for showing severe spondylolisthesis, perhaps as a result of the pelvic incidence and sacral slope.

    - In a second `LOF` run, a group of six other records with high scores was included. Of these, only patient `52` was identified as a pure multivariate outlier, as they did not present extreme values in any of the variables, except for `pelvic_tilt`, which shows a high value, although it cannot be considered extreme. Patient no. `52`, unlike patient no. `116`, does not stand out in any of the metrics and does not have an obvious diagnosis according to the data studied.

- Clustering-based methods

    - Using the k-means method (ordering outliers according to the Euclidean distance from the centroids), patients `116` and `198`, who had already been treated with other methods, were identified as outliers. Patients `76`, `96`, and `163` also appeared as outliers.

    - Patient `163` had fairly high values (without being very extreme) in several variables; the sum of the effects of these variables may have led to a high score.

    - In the case of patients `76` and `96`, although they only had extreme values in one variable, the fact that they had high (but not extreme) values in other variables may also have affected their scores.

