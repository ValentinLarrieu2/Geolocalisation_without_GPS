@author: Valentin Larrieu

# Geolocalisation without GPS

# Report

The goal of this document is to predict the position of a device (Longitude, Latitude) using the Data given. The information available are :

- The Id of the message sent by the device
- The Id of the Base station
- The Id of the device
- The nseq (~ number of sequence linked with the number of time the message have been sent before being heard by stations)
- The Rssi (~ power of the signal received)
- The time\_ux (~ moment at which the message is received by the station)
- The bs\_lat (~ latitude of the base station)
- The bs\_lng (~ longitude of the base station)

1. 1Dataset Exploration

The first things done was to see the repartition of the data. We did some plot of the data, grouped them by messages.

It allowed us to identify some key points :

- Some messages are received by a lot of stations ( ~ a hundred) but some only by one
- Most of the dataset is localized in the US ( the base stations)
- Some stations appears to be located far away from others (some even thousands of kilometers)
- The test set provides data of devices not seen in the train set

1. 2Test of the first model

We then tried our first model. The first one was given to us :

- The feature engineering creates one column per base station and one line per message Id. The result is a one on the line of the message if it is received by the station
- 2 models are build, one for each of the y (lat and lng) : LinearRegression with a crossval of 10
  - Precision at 80% : ~ 7km

1. 3Improving the model

## Model change

We then tried to improve the implemented model. Only changing the Linear Regression by a Random forest gave us a precision of almost 2km.

But here we identified a problem: the way the data was split was creating a problem : some messages of the test set were also in the train set thus this precision.

## Validation change

So, we changed the implementation to avoid this problem. We implemented our own cross validation. We created a « leave one device out » : we train on all the devices excepted one and test on the remaining one, then we iterate on all devices.

## Feature engineering

 We also created several features to improve our results. We added some column like (for each station):

- nseq
- rssi
- lat
- lng

## Trying new models

In parallel with that work, we tried others model, for instance:

- ExtratreesRegressor
- SVR
- RandomForestRegressor
- Linear regression

We then focused on the improvement of those models trying other parameters, with param grid on the cross val. We also tried boosting method such as adaboost to see if we could get better performances.

The best one was the simple RandomForestRegressor when we only changed the number of estimators

## Results

Depending of the seed, we got an error at 80% of approximately 6.3km with a RandomForestRegressor

# 4 New features engineering

## Problem identification

As we saw earlier, the repartition of the stations is not homogenous. Some are located thousands of kilometers away. That&#39;s why we tried to remove those station at first. But those stations sometimes are the only one to receive a message, so we can&#39;t exclude them if we want to predict correctly

We also identified that the leave one device out was creating an asymmetry: some devices have a lot of messages and are received by a lot of stations some just a few. So, it influences a lot the test for the leave one device out.

Moreover, we realized that the feature matrix could be difficult to train because it contains a lot of 0. So instead of having a four feature (rssi, nseq, lat, lng) for each station we decided to have four features for each of the 5 first station which received the message (we replicate the value if there is less than 5 station). With this matrix, each feature is useful for every prediction and consistent.

# 5 New models

## Find the best model

With the new features we tried different models like:

- Linear Regression
- Random Forest
- Neural network

As expect even with a lot of improvement on the neural networks, we failed to have good performance due to a lack of data. The error for the percentile 80 is around 8.9km.

However, we had a real improvement on the linear regression with the new feature. The result for the first feature matrix gives us prediction out of the range of the latitude ([-90, 90]), but with the new feature matrix we have a score up to 6.4km with a leave one device out.

The Random forest algorithm is the algorithm which have the best performance among the model we tried with a score (on percentile 80) of 5km with default parameters.

## Improving model and grid search

We improve the algorithm by tuning the parameters of the random forest model by using a grid search. We tune parameters one the following grid search:

- Max\_depth: [10, 20]
- N\_estimators: [50, 100, 300]

The best parameters were max\_depth = 20, n\_estimators=300.

## Final Result

With the Random Forest Regressor we obtain an error around 4.8km

