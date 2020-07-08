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


### Licensing, Authors, Acknowledgements<a name="licensing"></a>
MIT License








