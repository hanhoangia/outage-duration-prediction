# Predicts the Duration of an Outage that Caused by Severe Weather
Author: Han Hoang

This project is completed as an assignment for DSC 80 at UCSD.

### Framing the Problem
Our goal in this project is to predict the outage duration of an outage that is caused by severe weather. This is a regression type of machine learning problem because we are trying to predict a number for a continuous variable (i.e. OUTAGE.DURATION) using the dataset from [Purdue University's Laboratory for Advancing Sustainable Critical Infrastructure](https://engineering.purdue.edu/LASCI/research-data/outages). For a regression problems, there are 2 evaluation metrics we can use to evaluate our model performance: R squared (or R^2) and Root Mean Square Error (RMSE). R squared measures how well our model can capture the variance in the actual output, while RMSE is a number that indicates the average prediction error of our model.

### Baseline Model
To start off with our prediction task, we need to establish a baseline model to compare the performance against and improve upon. We picked our baseline model to be a regression model based on Random Forest, which based itself on Decision Tree, because Decision Tree is quick to train and easy to interpret. 

The predictors we use to train our model are POSTAL.CODE and PROPORTION.AFFECTED:

|                     | Description                                                           | Variable Type |
|---------------------|-----------------------------------------------------------------------|---------------|
| POSTAL.CODE         | The state the outage occurred in                                      | Nominal       |
| PROPORTION.AFFECTED | The proportion of customers that got affected by the outage in the US | Quantitative  |

*Note*: A nominal variable is a categorical variable that does not have an inherent order in and of itself.

Because a regressor only processes number input for number output and does not process categorical values, we need to convert POSTAL.CODE to number value by creating a column for each state value and put 1 for that column if the outage occured in that state, 0 otherwise. This conversion method is called one-hot encoding.


After we one-hot encoded POSTAL.CODE and trained our model, we got an RSME around 2976 and R^2 -0.0095. 

*Note*: The mean of the outage duration in the data is around 2932 and the standard deviation 2873.

The standard deviation is by definition a measurement of how far a number in the dataset is from the mean (i.e. the average), but our average prediction error is very close to the standard deviation of the actual data, which makes our model performance mediocre since it can barely restrict the range of our expected value in the dataset. In other words, our model prediction is as good as we randomly picking any point in the range of outage duration and make that our prediction.

Also, the R^2 is so close to 0 that it is on par with our interpretation of RSME that our model is very bad at capturing any meaningful pattern in the data.

### Final Model
To make an improvement on the baseline model, more features are added to the model. The full list of features used to train the model are now:

|                     | Description                                                           | Feature Relevancy                                                                                                                          |
|---------------------|-----------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| POSTAL.CODE         | The state the outage occurred in                                      | States that have outages often due to severe weather are likely to be more well-prepared and thus outage duration are likely to be shorter |
| OUTAGE.START.HOUR   | The hour the outage started to occur                                  | Outages started at peak hours or during the night likely have longer duration                                                              |
| CUSTOMERS.AFFECTED  | The number of customers that got affected by the the outage in the US | Number of customers got affected tells us how big the outages are; the bigger the outages, the longer the duration likely are              |
| PROPORTION.AFFECTED | The proportion of customers that got affected by the outage in the US | How much of the customer population got affected by the outages; another measurement of how big the outages are                            |
| MONTH               | The month the outage occurred in                                      | Some months like July tend to have more, bigger outages happening and is harder to deal with; thus can make the outage duration longer     |

Our final model is still a Random Forest regressor, but with added features and hyperparameters of the model tuned. Random Forest have a lot of parameters we can tune upon, but we picked out 3 hyperparameters to tune for our model and found the best estimator for each by testing how our model performs with every distinct combination of the 3 hyperparameters of different range of values using the method GridSearchCV:

|                   | Description                                                                        | Best Estimator |
|-------------------|------------------------------------------------------------------------------------|----------------|
| n_estimators      | The number of Decision Trees in a Random Forest                                    | 90             |
| max_depth         | The maximum depth of a Decision Tree in a Random Forest                            | 8              |
| min_samples_split | The minimum sample size required for a split in a Decision Tree of a Random Forest | 2              |

The model with the best 3 estimators as its hyperparameters have an RSME of around 2591 and R^2 of 0.235.

Because the final model's RSME is lower than the base model's RSME by about 360, and its R^2 higher than the base model's R^2 by about 0.228, we can conclude that our final model did improve upon the base model.


### Fairness Analysis
To test the fairness of our model, we want to conduct a permutation test to see if our model performs the same for 2 different groups of data. For this test, the first group X will be all of the outages where the outages started to occur during the AM hours (0-11) and the second group Y will be all of the outages where the outages started to occur during the PM hours (12-23).

- **Null Hypothesis**: The model's RSMEs for the data that contains outages that start in the AM hours are not different from the model's RSMEs that contains outages that start in the PM hours. Our model is fair.
- **Alternative Hypothesis**: The model's RSMEs for the data that contains outages that start in the AM hours are different from the model's RSME that contains outages that start in the PM hours not by chance alone. Our model is not fair.

Because we want to determine whether the distribution of group X and Y is any different, our evaluation metric is the absolute difference of their RSME. The signifiance level is conventionally 0.05.

<iframe src="assets/model-fairness-test.html" width=800 height=600 frameBorder=0></iframe>

Since the resulting p-value is 0.14 and is above our cutoff of 0.05, we failed to reject our null hypothesis, and we conclude that the model's RSMEs for the data that contains outages that start in the AM hours are not different from the model's RSMEs that contains outages that start in the PM hours. Our model is fair.



