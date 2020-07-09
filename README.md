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
To prevent customer attrition, it is important to identify those who are at risk of churn. In this project, we perform churn prediction on users' data log of Sparkify which is a fictional music streaming service created by Udacity. On Sparkify, users can upgrade, downgrade or cancel their subscription at any time. We define cancellation confirmations of service as churn events.

The full dataset consists of 12 GB of users' interaction logs. Udacity has also provided a mini dataset of 128 MB. The strategy for predicting churns includes performing exploratory data analysis and model developments using the mini dataset first, and then modifying these models so that they are scalable for the full dataset. These models are trained with full dataset on Amazon EMR platform using Apache Spark.

### File Descriptions<a name="file_descriptions"></a>
- Sparkify.ipynb: notebook for data exploration, feature engineering, modeling and evaluation with mini dataset
- Sparkify_aws.ipynb: notebook for modeling and evaluation with full dataset on Amazon EMR cluster
- images: graphs
- LICENSE.txt: MIT License

### Results<a name="results"></a>
#### Exploratory Data Analysis
There are 286500 records in the mini datatset. We find that there are missing values for registration. Users without registration may be guest users and are not valid for our churn analysis. We therefore drop the rows with missing registration values. After dropping, we still have missing values in `artist`, `length`, and `song`. However, a closer look at the `page` reveals that these columns do not have missing values when a user is on the page `NextSong`.

We define churn event as a user visiting `Cancellation Confirmation` page and identify users who are churn. We find that the labels are not balanced. Only about 16% of the labels are for churn users.
There are 255 distinct users and 52 of them are churn.

![churn](/images/churns.png)

#### Feature Engineering
We create aggregate features to gain more insight into the behavior of users who stayed versus users who churn. We use these aggregates to observe how much of a specific action they experienced such as number of thumbs downs or how many times they added new friends per week. The relations between users' attributes such as gender, level, location, and churn events are also explored.

![thumps_down](/images/num_Thumbs_Down.png)

There are 104 female users and 121 male users. Twenty female users and thirty-two male users are churn.

![gender](/images/gender.png)

There are 195 users whose subscription level is free and 165 paid users. We find that paid users churn more. We notice that these two groups add up to more than the total number of distinct users. The reason is because some users change their subscription level multiple times.

![level](/images/level.png)

Next, we extract user's state from location and find the total number of churn events in each state. The number of churn events vary across the states, and thus users' states can be a potential feature in predicting churn.

![state](/images/state.png)

How long a user has been with Sparkify can be found from the registration and timestamp. We find that users who stayed tend to be with the service longer than those who churn.

![registration_days](/images/days_since_registration.png)

We also find the number of churn events for the time of day. For this mini dataset there are no churn events between 3 AM and 4 AM. Similar analysis is done for days and months, and we find that there are no churn events in September and December.

![hourly_churn](/images/hourly_churn_events.png)
![monthly_churn](/images/montly_churn_events.png)

### Licensing, Authors, Acknowledgements<a name="licensing"></a>
MIT License

I thank [Udacity](https://www.udacity.com) for their support. 







