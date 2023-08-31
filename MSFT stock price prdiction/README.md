# Overview
The project was based on the Case 1 from the book "Machine Learning and Data Science Blueprints for finance" to predict Microsoft stock price. The goal was to beat the performance of proposed model and many reinforcments were added to achieve that. For example, more features like technical indicators and smoothing features were included to construct a better dataset and PCA was introduced to handle the overly large data. More related machine learning models like XGBoost and LSTM were used. In addition, to make the results more robust, different financial metrics such as CAGR, Sharpe Ratio were used to evaluate the performance and we conducted a final White Reality Check to validate the performance as well.

# Methods
### Data engineering

Stock, index adjusted close data was imported from yahoo finance. Kalman filter was calculated using pykalman and compared to standard moving averages.

- Comparison of Kalman Filter vs MA's:  (Notice how KF is more adaptive as it accounts for estimation errors and tries to adjust its predictions from the information it learned in the previous stage)
![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/772505f6-9408-440b-8433-afc181f98632)


More technical indicators including RSI, ADX, MACD, Money Flow Indicator, Momentum were added by using fcuntions from Talib library. Among them, Momentum，RSI show greater correlation with the Microsoft’ stock Price.

- Correlation matrix:
![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/aad08dbc-9fb6-4325-af9b-70e7a0ef6957)


### Feature selection

Since we add more predictors in the dataset, there’re 23 predictors in total. In order to reduce the dimensionality of this large dataset, we decide to use PCA to remove unnecessary component, while keeping variance at 95% level. 
We perform train-test split before operate PCA, making size of test set equals 20% dataset. 

PCA requires standardization since it is sensitive to the scale of inputs. Standardization is calculated on training set and later apply to the test set, in order to prevent using any information coming from the test set before or during training, which is a potential bias in the evaluation of the performance.

- The first 14 Principle Components explain 95%  variance of the data:
  
<img width="809" alt="e251dadc3fc0651d0ef1eb23fbc4549" src="https://github.com/pengp5781/Financial-Engineering/assets/111671117/69dbfa5b-6a04-4dd7-8ce9-4d4ae84d7fb1">

### Modeling
In the original problem, many classical models like logistic regression, Lasso regression and KNN were used.

In our experiment, in addition to the proposed models, XGBoost was added due to it being a boosting method with high performance,  good accuracy and scalability.

- Cross validation performance of models (scoring: neg_mean_squared_error):

 ![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/c71e5fa8-729d-48f1-ad22-a71c799424c1)

However, XGBoost has the same problem as other boosting methods. All boosting method has low train error but high test error. Overfitting problem exits in these models and should be avoided.
Ensemble models, such as gradient boosting regression (GBR) and random forest regression (RFR) have low bias but high variance.

- Train/test scores: 
![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/7c0efc0b-024d-46b2-990a-88d693907ff4)


Looking at the training and test errors, our new dataset illustrate different results from the original program: The LSTM model performs decently, which beats the ARIMA model in the test set. Hence, we select the LSTM model to move on to final model tuning.

### Model Tuning

Initially, it was proposed to use Gridsearch to find the optimal value for following hyper-parameters:

- Number of neurons 
- Learning rate
- Number of epochs
- Batch Size
- Optimizer 
- Regularizer
  
However, due to number of combinations of all the hyperparameters for tuning is extremely large, the gridsearch process took well over hours to complete.
So alternatively, we seperate the process into 2 steps to reduce run time:

- In the first step, we fix the all the hyperparameters except optimizer and regularizer. This would allow us to find the best optimizer and regularizer of this hyperparameter setting.
  
- In the second step, we use the tuned optimizer and regularizer from the first step to tune the remaining hyperparameters.

The total running time is reduced in a large extent, from more than 2 hours to only 30 minutes. 
It turns out that the best configuration is optimizer=Adamax, regularizer l1=0.01, Number of neurons =120, number of epochs=50, batch size=5, learning rate=0.001.

We first tried fitting our model using above configuration, the test results seemed to be pretty good: The MSE on test set decreased from 0.00081 to 0.00069. However, when we visualize the predictions, the predicted cumulated return we obtain had very small fluctuation compared to the actual data. We realized that this might because l1 regularizer is too strict in the grid search. Using L1 limit the scale of weight towards zero, causing predicted power impaired, and we ended up with a mild results.

<img width="899" alt="1693457481(1)" src="https://github.com/pengp5781/Financial-Engineering/assets/111671117/8e3cd51e-87ed-458e-8c13-789800788dd4">


# Conclusion
### Results
We calculated some evaluation metrics to measure the performance of the system 
This included financial metrics such as CAGR, Sharpe Ratio, Maximum Draw Down, and Calmar Ratio.
The result of theses metrics are shown below. 
- Compound annual growth rate on test set achieve 33% return on our trading system.
- Sharpe ratio is 1.83, which is greater than 1.0 indicating a high degree of expected return for a relatively low amount of risk

  <img width="890" alt="1693457661(1)" src="https://github.com/pengp5781/Financial-Engineering/assets/111671117/bea27a52-ac82-4868-816b-e4ef77f1f4bb">

We then plot the train and test equity curve based on cumulative log return on both training and test set. 
The result is shown below. 

<img width="899" alt="1693457963(1)" src="https://github.com/pengp5781/Financial-Engineering/assets/111671117/3379a48f-2f62-4908-b1fb-23e8b2c0815f">

### White Reality Test

We perform White’s Reality Check using detrended price data. The results is shown below:

![image](https://github.com/pengp5781/Financial-Engineering/assets/111671117/97ca08bd-6adf-49ee-ba1b-f78a63244ba3)

The p-value is 0.0736, statistically significant at 10% significance level.
We reject the null hypothesis that our system's return is zero.
Indicating that our algorithm has ability to predict the Microsoft stock price.


### Potential Bias and Possible Improvements

The test set is used when tuning and selecting optimal model in our program, which might making our results over optimistic.

The information from unseen data should be set aside in the training process, and should be only used when making predictions based on finalized model. 

Model from information leakage may do well on that particular data even if it can't generalize to new data. 

We should create validation set for training process, and only use test set when testing model’s ability to properly adapt to unseen data.
