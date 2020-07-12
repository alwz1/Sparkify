# Churn Prediction using Apache Spark on Amazon EMR

### Table of Contents
1. [Installations](#installations)
2. [Project Motivation](#project_motivation)
3. [File Descriptions](#file_descriptions)
4. [Results](#results)
5. [Licensing, Authors, Acknowledgements](#licensing)

### Installations<a name="installations"></a>
Amazon EMR 
- three m5.xlarge instances
- Apache Spark

### Project Motivation<a name="project_motivation"></a>
To prevent customer attrition, it is important to identify those who are at high risk of churn. In this project, we perform churn prediction for Sparkify, a fictional music streaming service created by Udacity. It has provided with a full dataset of 12GB and also a mini dataset of 128MB. 

The strategy for predicting churns includes performing exploratory data analysis and model development using the mini dataset first, and then adapting models to make them scalable for the full dataset. The adapted models are trained on Amazon EMR platform with full dataset using Apache Spark.

A blog post of this project is available on [Medium](https://medium.com/p/c2ca21be798b/edit).

### File Descriptions<a name="file_descriptions"></a>
- Sparkify.ipynb: notebook for data exploration, feature engineering, modeling and evaluation with mini dataset
- Sparkify_aws.ipynb: notebook for modeling and evaluation with full dataset on Amazon EMR cluster
- images: graphs
- LICENSE.txt: MIT License

### Results<a name="results"></a>
#### Exploratory Data Analysis
Initial data exploration is performed using the mini dataset. No additional information on the columns is provided, but most column names are self-explanatory. They can also be understood with a little exploration - `level` for user's subscription tier, `registration` and `ts` for timestamps, and `status` for HTTP status codes. User interactions with the service such as logging in, playing next song, and adding friends can be found in page.

There are 286500 records in the mini dataset, and 8346 missing values in registration. Users with missing registration values may be guest users and are not relevant for churn prediction. Therefore, rows without registration values are dropped. After dropping missing registration values, there are still 50046 missing values in artist, length, and song. These missing values result from users visiting other pages instead of playing the next song; they do not have missing values when a user is on `NextSong`.

Churn event is defined as a user visiting `Cancellation Confirmation` page. `churn_event` and `churn_use` columns are then created. It is observed that the classes are not balanced. Only about 16% of the labels are for churn users.

There are 255 distinct users and 52 of them are churn.

![churn](/images/churns.png)

#### Feature Engineering
To gain more insight into the behavior of users who stayed versus users who churn, aggregate features are created. These aggregates are used to observe how much of a specific action they experienced such as number of thumbs downs or how many times they added new friends per week. The relations between users' attributes such as gender, level, location, and churn events are also explored.

![thumps_down](/images/num_Thumbs_Down.png)

There are 104 female and 121 male users of which 20 female and 32 male users are churn. The percentage of churn by gender can be seen below.

![gender](/images/gender.png)

There are 195 users whose subscription level is free and 165 paid users. It is found that paid users churn more. The total number of free and paid users is more than that of distinct users because some users change their subscription level multiple times.


![level](/images/level.png)

Next, `state` for each user is extracted from `location`, and the distribution of churn events by state is obtained. The number of churn events vary across the states, and therefore it can be a potential feature in predicting churn.

![state](/images/state.png)

Subscription lengths for users are found from `registration` and timestamp `ts`. It is observed that users who stayed tend to be with the service longer than those who churn.

![registration_days](/images/days_since_registration.png)

Next, seasonality of churn events is investigated. For the mini dataset there are no churn events between 3 AM and 4 AM and also none in September and December.

![hourly_churn](/images/hourly_churn_events.png)
![monthly_churn](/images/monthly_churn_events.png)

The following thirteen features are selected as potential features for model development. The correlations between the features are also checked to avoid multicollinearity.

![features](/images/features.png)

The absolute values of the correlation coefficients between selected features of mini dataset are shown in the following heatmap. It is observed that `month` has the highest absolute correlation coefficient of 0.27 with the label churn_user followed by `days_since_registration` with 0.23.

![heatmap](/images/heatmap.png)


#### Data Preprocessing
Data preprocessing steps for the mini dataset are following. 
- Clean data by removing rows with missing registration values.
- Add churn event and churn user columns.
- Add number of days since registration.
- Add hour, day, and month columns. 
- Add aggregate feature columns: num_Thumbs Down, num_Roll Advert, num_Add Friend, and num_Error.
- Extract users' states from location. 
- Remove unnecessary columns. 

For the full dataset we make the following modifications to data preprocessing. 
- Instead of using itemInSession directly, find the average number of items in session for each user. 
- Find the maximum number of days since registration for each user. 
- Find the average hour, day and month that the user is active.
- Add phase of subscription level for each user.
- Remove duplicate rows. 

These modifications help keep the size of the preprocessed data in check to be trained efficiently on the AWS cluster. 
The labels of the preprocessed data are imbalanced with about 23% of the labels for churn users.
We also prepare machine learning pipelines for logistic regression, random forest and gradient boosted trees models. Stages for the pipelines include indexing and one-hot encoding categorical columns, transforming columns into a single vector column, and scaling the values of the features into the range from 0 to 1.

#### Modeling and Evaluation
##### Mini Dataset
We train logistic regression, random forest and gradient boosted trees models. The dataset is split into 80% train and 20% test datasets. The Spark ML pipelines are instantiated, and the models are first trained with default parameters for the classifiers. 

Since we have imbalanced dataset, accuracy is not the best metrics to evaluate models' performance. Instead, we use weighted F1 scores as the metrics for evaluating performance of the models. In addition, we find AUC scores, area under ROC curve. The models are evaluated using the test dataset. For the mini dataset using the default parameters for the classifiers, we obtain the following results.

![f1_auc_mini](/images/f1_auc_mini.png)

We have the best F1 score with the gradient boosted trees model. This model's hyper-parameters, `stepSize` and `maxIter` are then tuned using a three-fold cross validation on the training dataset. We have the best F1 score of 0.98 for the mini dataset with `stepSize = 0.5` and `maxIter = 32`. 

##### Full Dataset

Before the data preprocessing was adapted to include more aggregates by userId as mentioned in the previous section, training the gradient boosted trees model with full dataset on Amazon EMR cluster was taking a long time and would eventually time out. We tried increasing Spark driver memory, but the problem persisted. Preprocessing the full dataset with more aggregate features solved this problem. 

On Amazon EMR cluster, we trained logistic regression, random forest and gradient boosted trees models. The cluster consisted of three nodes with instance type of m5.xlarge. The full dataset of 12GB was split into 80% train and 20% test datasets. Model performance was evaluated on test dataset. We used default parameters for logistic regression and random forest classifiers. For GBTClassifier we used `maxDepth = 7`, `maxIter = 50`, `stepSize = 0.8`, and `subsamplingRate = 0.8`. The choice of these parameters was guided by our experience in training with mini dataset and available resources. We achieved excellent results with GBTClassifier with weighted F1 score of 0.97 and AUC of 0.98. 

![f1_auc_full](/images/f1_auc_full.png)

We also checked for precision and recall for our best model. For predicting churn users, we obtained precision of 0.97 and recall of 0.90.

![classification_report](/images/classification_report.png)

![ROC](/images/ROC_Curve.png)



While we already have a very good performance with our current model, there are still potential improvements. We could extract region from location and use it as a feature instead of state, use different time windows for creating aggregate features, and perform more hyperparameter tuning with grid search.




### Licensing, Authors, Acknowledgements<a name="licensing"></a>
MIT License

I thank [Udacity](https://www.udacity.com) for their support. 







