Title: Final Project Report
Date: 11/24/2021   
Author: Obliviate
Slug: Final_Project_Report
Category: Final Project Report

# Prediction of Patient's Risk of Death based on Lab Test after Entering ICU


Team Name: Obliviate  
Team Members: Ruqian Cheng, Mingxuan Wang, Ruiqi Zhang


Product Web: [https://streamlit-app823.herokuapp.com/](https://streamlit-app823.herokuapp.com/)      
Project Blog: [https://obliviate823.github.io/](https://obliviate823.github.io/)      
Github Repository: [https://github.com/Obliviate823/823_FinalProject](https://github.com/Obliviate823/823_FinalProject)      

<br/>

## Team Part


### 1. Project Purpose

The target user of this project is hospitals. For patients who enter the ICU, the risk of death is predicted based on the test values of their laboratory items and the number of days that the ICU stays. For people who are at high risk of death, the hospital should focus on the changes in their vital signs and take preventive measures.

### 2. Data Processing

This project uses MIMIC III as the main dataset. We choose the variable “hospital_expire_flag” in ADMISSIONS.csv file as target variable, and we want to predict the death risk for patients who enter ICU for the first time. Therefore, our main purpose in data processing part is to find variables that related to the target variable for the next part of data analysis. To find these variables, we use five files: “LABEVENTS.csv”, “ICUSTAYS.csv”, “ADMISSIONS.csv”, “D_LABITEMS.csv”, “PATIENTS.csv”.  

**Dataset Introduction**

ICUSTAYS: Including the ICU information for each patient.

LABEVENTS: Including the lab test results for each patient. 

ADMISSIONS: Including the hospital information for each patient and whether patient died in this hospitalization.

PATIENTS: Including basic information such as birthday and gender.

D_LABITEMTS: Including the actual lab item name for each item id.

Firstly, we obtain the intime, outtime and length of stay for each patient from the dataset “ICUSTAYS.csv”. Many patients enter ICU multiple times, and we only choose the data when the patient enters the ICU for the first time.

Secondly, we want to implement labevents processing. Each row of the datasets represents the value of a certain item tested by a certain patient. A patient may have multiple tests for one item. For a certain item, we want to obtain the value of the patient's first test during the first time entering ICU. After processing, for each patient and each item, there is only one row. Therefore, we can convert the dataset by “pd.pivot”. After converting, delete inappropriate data and missing values.

As for admissions dataset and patients dataset, through subject_id, we obtain the patient admission time, gender, and birthday, then obtain the age at admission time. Then combine these data to coverted labevents dataset through subject_id.

But there are still too many variables. Therefore, we use simple logistic regression to fit a model, and select the top 10 important variables for the next data analysis part.

After processing, we obtain a dataset “death_risk_predict.csv”, including subject_id, 10 variables and 1 target variable. There are 17809 rows, corresponding 17809 patients.

### 3. Model Training

**First Part**

For the first part of model training. We select logistic regression and SVM model. Because the data is unbalanced, only about 17% of the data has 1 outcome, we choose AUC as our model evaluation. After normalize the data, the logistic regression has 0.77 AUC.
For the SVM model, we try three kernels: Linear kernel, polynomial kernel and rbf kernel. After comparison, the polynomial has the best AUC, which is close to 0.79. Therefore, finally we choose polynomial kernel SVM model to predict the death risk of patients.


**Second Part**

We used bagging and boosting technique to build the classification model.

Firstly, we built a random forest model because it uses bootstrap sampling and consider randomly chosen features when splitting to make trees diverse and reduces variance. 

After we built the model, we used cross-validated grid search with AUC as the evaluation metric to tune the parameters. With the usage of the function GridSearchCV, we performed a 5-fold cross validation on the training set and enumerate all the possible parameters in a selected range to find the best parameter. A function, show_gridsearch_result(), is written, to plot the result of grid search. Since the dataset is relatively large and random forest is computationally intensive, the parameters are tuned with a “greedy method” – tune one of the parameters at a time. In random forest, when n_estimators (the number of trees in the random forest) increases, the AUC of the model is nearly monotonically increasing, so we tune n_estimators first. The AUC score becomes stable since n_estimators=400, then slightly increases as the n_estimators increases. To secure a high auc while avoid the problem of consuming too much training time, n_estimators=400 was chosen. Then, we tuned tree-depth related parameters. Since parameter 'max_depth', 'min_samples_split', 'min_samples_leaf' are all parameters related to the depth of the tree, they are correlated. Therefore, these 3 parameters were tuned together using grid search. Then, we tuned max_features, the number of features considered when splitting. To calculate AUC and plot POC, we wrote a function show_auc_roc() to do that. After tuning, the AUC of the model increases from 0.743 to 0.765.

SHAP is used to interpret the model. Urea Nitrogen, Potassium are pushing the prediction higher than the average model based on the training set. Anion Gap, RDW, Bases Excess, White Blood Cells, icu_los, Sodium, SUBJECT_ID are pushing the prediction lower.

Random forest requires a lot of computational power and resources, so we decided to use a XGBoost model to reduce the time of model training because it uses the power of parallel processing within each tree when creating branches and is much faster than random forest. 

During parameter tuning, we also used cross-validated grid search with AUC as the evaluation metric. We tuned n_estimators, which is the number of boosting rounds, firstly. Then, since max_depth, Maximum tree depth, and min_child_weight, Minimum sum of instance weight needed in a child, are correlated, they are tuned together. Learning_rate, a parameter to slow down the training is tuned to avoid overfit. Finally, we build the final model and report ROC curve.

The AUC of XGBoost is less than random forest. This can be explained because according to exploratory data analysis, our dataset contains a lot of outliers. Since our XGBoost uses a serial process to train each base classifier based on the previous residual, the effect of these outliers on the model will be increased; thus, it will doesn’t perform well as the random forest model.

By using SHAP, the model is interpreted. Urea Nitrogen, Potassium, Chloride are pushing the prediction higher than the average model based on the training set. RDW, Anion Gap, Bases Excess, Sodium, White Blood Cells, icu_los are pushing the prediction lower, which is much similar with the random forest model. 


### 4. Web Product

`Streamlit` is used to build the web apps for machine learning results and interact with users. The information of blood test variables is listed. If users want to predict the risk of death for patients, they need to input blood test results by scrolling the slider. After inputing, the prediction outcome will be shown on the right. In addition, there is a select box at the bottom right. Users can select the model they are interested in to understand the ROC curves and AUC values of different models.

<br/>




## Individual Part 

### Ruqian Cheng

I participated in the selection of data sets and machine learning algorithms, and was mainly responsible for the web application part, using streamlit to build a web dashboard and deploying the product to a cloud platform. Through this project, I become more familiar with various machine learning algorithms and mastered the construction of a web interactive platform.


### Mingxuan Wang

Personally, I built the random forest and the XGBoost model, including tuning the parameter, reporting the AUC and ROC curve, and using SHAP to interpret the model. Moreover, I built the dashboard on web using streamlit to transform our model to a online machine learning model that can be used to predict the probability of death by entering patient’s lab data. 

When doing this project, I had a better understanding of the machine learning algorithms, especially bagging and boosting, by building models in a practical way. I found the process of tuning the parameters interesting because it help understanding the algorithm. For example, I learned Adaboost before, but I have no experience with XGBoost. By doing this project, I learned the algorithm of XGBoost as well as GDBT online, which I believe will benefit me a lot in the future. 

Moreover, I learned from the experience that the dataset affects a lot to the algorithm so we should choose algorithm based on our data. After the two models are built, I am surprised that the random forest model has the higher AUC than the XGBoost model because XGBoost usually works better. The reason of that became clear when I recalled the result of our exploratory data analysis: there are a lot of outliers in our data set and XGBoost is very sensitive to outliers. This helped me to understand that we need to choose our model based on the characteristics of our data set and no model always works better than others.

In the process of building data dashboard, I learned to build a interactive dashboard in a web application. I learned to think from the view of the users: what layout and interactive buttons will improve their user experiences? By creating dashboards, I had some basic exposure to build interactive applications. For example, the situation where users are inputting some invalid strings need to be considered when building the interactive dashboard. It should be excluded or be fixed by using slide bars or dropdown menu. This process catches my interest and makes me want to dive into it.

### Ruiqi Zhang

I am mainly responsible for data processing, the first part of model training and blog construction. In the data processing part, I find that for real data, it is very difficult to get a data set that is highly relevant. I try a lot of variables, and most of them has a very poor performance in modeling training. In the training of models, I learned more about the theory of SVM models.