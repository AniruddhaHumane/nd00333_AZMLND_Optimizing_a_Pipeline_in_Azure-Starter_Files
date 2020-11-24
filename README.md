# Optimizing an ML pipeline in Azure

## Summary of the problem statement

In this project, we want to create and optimize ML pipeline. To do this, we are using the [UCI Machine Learning Repository: Bank Marketing Data Set](https://archive.ics.uci.edu/ml/datasets/Bank+Marketing) to predict whether the client will subscribe a term deposit. 

## How the problem was solved

We are creating a tabular dataset from the link mentioned. After preprocessing we are using it to train a sci-kit learn logistic regression model and then we are optimizing its hyperparemeters `C` and `max-iter` through Azure ML's Hyperdrive. After finding the most optimal parameters, we are using the exact same dataset and running an AutoML experiement. AutoML then tries to identify which algorithm provide best accuracy with given configuration. After these results we will compare and evaluate which process provides beter models.

## Explanation of various factors

### Data
The data is taken from the UCI repository: [UCI Machine Learning Repository: Bank Marketing Data Set](https://archive.ics.uci.edu/ml/datasets/Bank+Marketing). 


#### Attribute Information:

Input variables:  
bank client data:  
1 - age (numeric)  
2 - job : type of job (categorical: 'admin.','blue-collar','entrepreneur','housemaid','management','retired','self-employed','services','student','technician','unemployed','unknown')  
3 - marital : marital status (categorical: 'divorced','married','single','unknown'; note: 'divorced' means divorced or widowed)  
4 - education (categorical: 'basic.4y','basic.6y','basic.9y','high.school','illiterate','professional.course','university.degree','unknown')  
5 - default: has credit in default? (categorical: 'no','yes','unknown')  
6 - housing: has housing loan? (categorical: 'no','yes','unknown')  
7 - loan: has personal loan? (categorical: 'no','yes','unknown')  

related with the last contact of the current campaign:  
8 - contact: contact communication type (categorical: 'cellular','telephone')  
9 - month: last contact month of year (categorical: 'jan', 'feb', 'mar', ..., 'nov', 'dec')  
10 - day_of_week: last contact day of the week (categorical: 'mon','tue','wed','thu','fri')  
11 - duration: last contact duration, in seconds (numeric). Important note: this attribute highly affects the output target (e.g., if duration=0 then y='no'). Yet, the duration is not known before a call is performed. Also, after the end of the call y is obviously known. Thus, this input should only be included for benchmark purposes and should be discarded if the intention is to have a realistic predictive model.  

other attributes:  
12 - campaign: number of contacts performed during this campaign and for this client (numeric, includes last contact)  
13 - pdays: number of days that passed by after the client was last contacted from a previous campaign (numeric; 999 means client was not previously contacted)  
14 - previous: number of contacts performed before this campaign and for this client (numeric)  
15 - poutcome: outcome of the previous marketing campaign (categorical: 'failure','nonexistent','success')  

social and economic context attributes  
16 - emp.var.rate: employment variation rate - quarterly indicator (numeric)  
17 - cons.price.idx: consumer price index - monthly indicator (numeric)  
18 - cons.conf.idx: consumer confidence index - monthly indicator (numeric)  
19 - euribor3m: euribor 3 month rate - daily indicator (numeric)  
20 - nr.employed: number of employees - quarterly indicator (numeric)  
  
Output variable (desired target):  
21 - y - has the client subscribed a term deposit? (binary: 'yes','no')

Note: The above information about the features is taken from [UCI Machine Learning Repository: Bank Marketing Data Set](https://archive.ics.uci.edu/ml/datasets/Bank+Marketing)



### Architecture
The  architecture of the solution is as follows:
![System Architecture](https://video.udacity-data.com/topher/2020/September/5f639574_creating-and-optimizing-an-ml-pipeline/creating-and-optimizing-an-ml-pipeline.png "System Architecture")
We are fetching the dataset from this [link](https://archive.ics.uci.edu/ml/datasets/Bank+Marketing) and creating a tabularDataset using tabularDatasetFactory. Once we have the tabular dataset factory we will divide our progress in two ways. 

First, We will train a sci-kit learn logistic regression model and optimize its hyperparameters using Azure ML's Hyperdrive. We will perform several iterations of this while randomly sampling values of `C` and `max-iter` from the hyperdrive configuration. After training we will note down its results. Then we will use the same dataset, create an AutoML configuration and submit the experiment. The AutoML experiement will then go through various models with a number of hyperparameters and try to identify the best model that can most accurately predict the outcome.

### Algorithm and Hyperparameters
The model that we are training through Hyperdrive is the sci-kit learn Logistic Regression. Logistic regression is one of the supervised classification machine learning algorithms where we take the input and try to classify it into two outcomes. Some of the examples of usage of logistic regresiona are, dog classification, yes or no problems, will a student pass or fail in an exam, etc. 

The hyperparameter's we will be optimizing are `C` and `max-iter`.

**C** 
float, default=1.0
This paremeter determines the inverse of the regularization strength. Smaller the value of this parameter, bigger is the regularization.
In my approach, I have uniformly selected the value of c from 0 to 1.

**max_iter**
int, default=100
This paremeter determines the maximum amount of iterations to do before converging.
In my approach, I am randomly selecting a number from 0 to 100.


### Why RandomParameterSampler?
This class is used to define random sampling over the search space of the hyperparemeter we are trying to optimize. The random sampling statistically provides much better search results and require less resources. I ran the experiement multiple times using random sampling and each time it converged quickly with the same accuracy.

### Why BanditPolicy?
BanditPolicy first trains a model and identifies its accuracy. Once it has a base value, it trains the next model. If the accuracy of the new model is not within the slack factor specified then it cancels the run. If the accuracy is better, then the accuracy of the new model becomes the new benchmark. This allows us to eliminate all the lower accuracy models that are not performing well. 


## Comparison of the outputs
### Hyperdrive

 - Finished much quickly as it was training only one model over the search space of hyperparemeters
 - Using Hyperdrive, sci-kit learn's Logistic regression achieved 90.74% accuracy.
 - Most optimal values for the hyperparemters we optimized are
	 - C - 0.2973202492447354
	 - max-iter - 16
 - It looks like only a maximum of 16 iterations was enough to produce an accuracy of 90% with an inverse regularization of 0.297.

### AutoML

 - AutoML training process took longer as it trained more than 50 models!
 - Out of all the models AutoML trained, the ensamble techniques like voting ensables and stack ensables outperformed every other model including the most optimal logistic regression model we achieved through Hyperdrive.

## Scope of improvement

 - Please cite the original dataset in the project repository
 - Please specify the problem that we are trying to solve instead of just specifying the dataset directly. The Azure's Hyperdrive and AutoML are techniques to solve such problems and are not the problem itself.
 - We can try any machine learning algorithms other than logistic regression and see how the data behaves.
 - We can try other preprocessing techniques including the clean_data function in train.py
 - Instead of Random Sampling we can try using other sampling techniques and see how that results with respect to random sampling.

## Proof of Cluster Cleanup
I deleted the cluster from the ML Studio's compute section.
![clean](./clean.png "Cluster deletion")
