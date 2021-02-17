---
title: "KKBox Churn Prediction"
date: 2021-02-15
tags: [churn prediction, data science, music streaming]
header:
  image: "/images/kkbox/kkbox.png"
excerpt: "Churn Prediction, Music Streaming, Data Science"
mathjax: "true"
---



## Overview

KKBOX is Asia’s leading music streaming service, holding the world’s most comprehensive Asia-Pop music library with over 30 million tracks. They offer a generous, unlimited version of their service to millions of people, supported by advertising and paid subscriptions. This delicate model is dependent on accurately predicting churn of their paid users.

In this project, we will build an algorithm that predicts whether a user will churn after their subscription expires. Here, we will determine that churn has occured if the customer has not renewed their subscription 30 days after it has expired or has been cancelled. Currently, the company uses survival analysis techniques to determine the residual membership life time for each subscriber. By adopting different methods, KKBOX anticipates they’ll discover new insights to why users leave so they can be proactive in keeping users dancing.

You can view the jupter notebook link [here] [data_analysis].

[data_analysis]: https://github.com/theperpetualmeatball/kkbox_churn_prediction/blob/main/Churn%20Prediction%20Model.ipynb


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sn
import time


from sklearn.model_selection import cross_val_score

#import models
from sklearn.linear_model import LogisticRegression
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis, QuadraticDiscriminantAnalysis
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
from sklearn.naive_bayes import GaussianNB
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import BaggingClassifier, AdaBoostClassifier, GradientBoostingClassifier, RandomForestClassifier, ExtraTreesClassifier
import xgboost as xgb
from xgboost import XGBClassifier

#dimension reduction
from sklearn.decomposition import PCA

# Feature Scaling for modelling purpose by using both min-max-scaling
from sklearn.preprocessing import MinMaxScaler
```


```python
s=time.time()
train = pd.read_csv('input/train_v2.csv') #contains the true churn values
print("Finished importing train in "+str(round(time.time()-s,2)))
sample_submission = pd.read_csv('input/sample_submission_v2.csv') #only relevent for kaggle submission purposes
print("Finished importing sample_submission in "+str(round(time.time()-s,2)))
transactions = pd.read_csv('input/transactions_v2.csv') #contains transaction data
print("Finished importing transactions in "+str(round(time.time()-s,2)))
user_logs = pd.read_csv('input/user_logs_v2.csv') #contains intra day data related to usage
print("Finished importing user_logs in "+str(round(time.time()-s,2)))
members = pd.read_csv('input/members_v3.csv')
print("Finished importing members in "+str(round(time.time()-s,2)))
```

    Finished importing train in 0.73
    Finished importing sample_submission in 1.47
    Finished importing transactions in 2.87
    Finished importing user_logs in 21.29
    Finished importing members in 27.34



```python
train.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>msno</th>
      <th>is_churn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ugx0CjOMzazClkFzU2xasmDZaoIqOUAZPsH1q0teWCg=</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>f/NmvEzHfhINFEYZTR05prUdr+E+3+oewvweYz9cCQE=</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>zLo9f73nGGT1p21ltZC3ChiRnAVvgibMyazbCxvWPcg=</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>8iF/+8HY8lJKFrTc7iR9ZYGCG2Ecrogbc2Vy5YhsfhQ=</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>K6fja4+jmoZ5xG6BypqX80Uw/XKpMgrEMdG2edFOxnA=</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
user_logs.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>msno</th>
      <th>date</th>
      <th>num_25</th>
      <th>num_50</th>
      <th>num_75</th>
      <th>num_985</th>
      <th>num_100</th>
      <th>num_unq</th>
      <th>total_secs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>u9E91QDTvHLq6NXjEaWv8u4QIqhrHk72kE+w31Gnhdg=</td>
      <td>20170331</td>
      <td>8</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>21</td>
      <td>18</td>
      <td>6309.273</td>
    </tr>
    <tr>
      <th>1</th>
      <td>nTeWW/eOZA/UHKdD5L7DEqKKFTjaAj3ALLPoAWsU8n0=</td>
      <td>20170330</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>9</td>
      <td>11</td>
      <td>2390.699</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2UqkWXwZbIjs03dHLU9KHJNNEvEkZVzm69f3jCS+uLI=</td>
      <td>20170331</td>
      <td>52</td>
      <td>3</td>
      <td>5</td>
      <td>3</td>
      <td>84</td>
      <td>110</td>
      <td>23203.337</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ycwLc+m2O0a85jSLALtr941AaZt9ai8Qwlg9n0Nql5U=</td>
      <td>20170331</td>
      <td>176</td>
      <td>4</td>
      <td>2</td>
      <td>2</td>
      <td>19</td>
      <td>191</td>
      <td>7100.454</td>
    </tr>
    <tr>
      <th>4</th>
      <td>EGcbTofOSOkMmQyN1NMLxHEXJ1yV3t/JdhGwQ9wXjnI=</td>
      <td>20170331</td>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>112</td>
      <td>93</td>
      <td>28401.558</td>
    </tr>
  </tbody>
</table>
</div>




```python
members.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>msno</th>
      <th>city</th>
      <th>bd</th>
      <th>gender</th>
      <th>registered_via</th>
      <th>registration_init_time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Rb9UwLQTrxzBVwCB6+bCcSQWZ9JiNLC9dXtM1oEsZA8=</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>11</td>
      <td>20110911</td>
    </tr>
    <tr>
      <th>1</th>
      <td>+tJonkh+O1CA796Fm5X60UMOtB6POHAwPjbTRVl/EuU=</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>7</td>
      <td>20110914</td>
    </tr>
    <tr>
      <th>2</th>
      <td>cV358ssn7a0f7jZOwGNWS07wCKVqxyiImJUX6xcIwKw=</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>11</td>
      <td>20110915</td>
    </tr>
    <tr>
      <th>3</th>
      <td>9bzDeJP6sQodK73K5CBlJ6fgIQzPeLnRl0p5B77XP+g=</td>
      <td>1</td>
      <td>0</td>
      <td>NaN</td>
      <td>11</td>
      <td>20110915</td>
    </tr>
    <tr>
      <th>4</th>
      <td>WFLY3s7z4EZsieHCt63XrsdtfTEmJ+2PnnKLH5GY4Tk=</td>
      <td>6</td>
      <td>32</td>
      <td>female</td>
      <td>9</td>
      <td>20110915</td>
    </tr>
  </tbody>
</table>
</div>




```python
transactions.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>msno</th>
      <th>payment_method_id</th>
      <th>payment_plan_days</th>
      <th>plan_list_price</th>
      <th>actual_amount_paid</th>
      <th>is_auto_renew</th>
      <th>transaction_date</th>
      <th>membership_expire_date</th>
      <th>is_cancel</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>++6eU4LsQ3UQ20ILS7d99XK8WbiVgbyYL4FUgzZR134=</td>
      <td>32</td>
      <td>90</td>
      <td>298</td>
      <td>298</td>
      <td>0</td>
      <td>20170131</td>
      <td>20170504</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>++lvGPJOinuin/8esghpnqdljm6NXS8m8Zwchc7gOeA=</td>
      <td>41</td>
      <td>30</td>
      <td>149</td>
      <td>149</td>
      <td>1</td>
      <td>20150809</td>
      <td>20190412</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>+/GXNtXWQVfKrEDqYAzcSw2xSPYMKWNj22m+5XkVQZc=</td>
      <td>36</td>
      <td>30</td>
      <td>180</td>
      <td>180</td>
      <td>1</td>
      <td>20170303</td>
      <td>20170422</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>+/w1UrZwyka4C9oNH3+Q8fUf3fD8R3EwWrx57ODIsqk=</td>
      <td>36</td>
      <td>30</td>
      <td>180</td>
      <td>180</td>
      <td>1</td>
      <td>20170329</td>
      <td>20170331</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>+00PGzKTYqtnb65mPKPyeHXcZEwqiEzktpQksaaSC3c=</td>
      <td>41</td>
      <td>30</td>
      <td>99</td>
      <td>99</td>
      <td>1</td>
      <td>20170323</td>
      <td>20170423</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# check the number of duplicate accounts in each table
print(train.duplicated('msno').sum())
print(sample_submission.duplicated('msno').sum())
print(transactions.duplicated('msno').sum())
print(user_logs.duplicated('msno').sum())
print(members.duplicated('msno').sum())
```

    0
    0
    233959
    17292468
    0



```python
#check number of columns and rows for each df
print(train.shape)
print(sample_submission.shape)
print(transactions.shape)
print(user_logs.shape)
print(members.shape)
```

    (970960, 2)
    (907471, 2)
    (1431009, 9)
    (18396362, 9)
    (6769473, 6)


Transactions and user_logs contain duplicate msno values because they provide granular data for each user per day. We will need to group the data by the msno column (user id) in order to merge it with the rest of the data.


## Explore feature correlation to target variable (is_churn)

Let's begin by briefly taking a look at how each of our features are correlated with the target variable. A low correlation does not necessarily mean that we should remove the feauture becuase it is not an accurate metric for classification models which only predict binary values. 


```python
#explore members
members.merge(train, on='msno',how='inner').corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>bd</th>
      <th>registered_via</th>
      <th>registration_init_time</th>
      <th>is_churn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>city</th>
      <td>1.000000</td>
      <td>0.513068</td>
      <td>0.018996</td>
      <td>-0.361908</td>
      <td>0.081424</td>
    </tr>
    <tr>
      <th>bd</th>
      <td>0.513068</td>
      <td>1.000000</td>
      <td>0.067176</td>
      <td>-0.393429</td>
      <td>0.065719</td>
    </tr>
    <tr>
      <th>registered_via</th>
      <td>0.018996</td>
      <td>0.067176</td>
      <td>1.000000</td>
      <td>-0.382191</td>
      <td>-0.080642</td>
    </tr>
    <tr>
      <th>registration_init_time</th>
      <td>-0.361908</td>
      <td>-0.393429</td>
      <td>-0.382191</td>
      <td>1.000000</td>
      <td>-0.000021</td>
    </tr>
    <tr>
      <th>is_churn</th>
      <td>0.081424</td>
      <td>0.065719</td>
      <td>-0.080642</td>
      <td>-0.000021</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



Since transactions and user_logs contain duplicate user id rows, we need to first group by the msno column (user ids) to synthesize the data into useful results. 


```python
# returns the max value of numerical variables and membership_expire_date
# returns the min value of transaction date
# returns the mode of ordinal variable and dummy variables, if multiple values share the same frequency, keep the first one
transactions_v2 = transactions.groupby('msno', as_index = False).agg({'payment_method_id': lambda x:x.value_counts().index[0], 'payment_plan_days': 'max', 'plan_list_price': 'max',
                                       'actual_amount_paid': 'max', 'is_auto_renew': lambda x:x.value_counts().index[0], 'transaction_date': 'min', 'membership_expire_date': 'max',
                                       'is_cancel': lambda x:x.value_counts().index[0]})

```


```python
transactions_v2.merge(train, on='msno',how='inner').corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>payment_method_id</th>
      <th>payment_plan_days</th>
      <th>plan_list_price</th>
      <th>actual_amount_paid</th>
      <th>is_auto_renew</th>
      <th>transaction_date</th>
      <th>membership_expire_date</th>
      <th>is_cancel</th>
      <th>is_churn</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>payment_method_id</th>
      <td>1.000000</td>
      <td>-0.264237</td>
      <td>-0.326858</td>
      <td>-0.325411</td>
      <td>0.326347</td>
      <td>-0.043490</td>
      <td>-0.127438</td>
      <td>-0.020496</td>
      <td>-0.203389</td>
    </tr>
    <tr>
      <th>payment_plan_days</th>
      <td>-0.264237</td>
      <td>1.000000</td>
      <td>0.965855</td>
      <td>0.965449</td>
      <td>-0.361611</td>
      <td>-0.025068</td>
      <td>0.656398</td>
      <td>0.005977</td>
      <td>0.455526</td>
    </tr>
    <tr>
      <th>plan_list_price</th>
      <td>-0.326858</td>
      <td>0.965855</td>
      <td>1.000000</td>
      <td>0.999404</td>
      <td>-0.394782</td>
      <td>-0.034520</td>
      <td>0.647513</td>
      <td>0.017064</td>
      <td>0.447071</td>
    </tr>
    <tr>
      <th>actual_amount_paid</th>
      <td>-0.325411</td>
      <td>0.965449</td>
      <td>0.999404</td>
      <td>1.000000</td>
      <td>-0.394979</td>
      <td>-0.034559</td>
      <td>0.647361</td>
      <td>0.016842</td>
      <td>0.444360</td>
    </tr>
    <tr>
      <th>is_auto_renew</th>
      <td>0.326347</td>
      <td>-0.361611</td>
      <td>-0.394782</td>
      <td>-0.394979</td>
      <td>1.000000</td>
      <td>-0.009964</td>
      <td>-0.184528</td>
      <td>0.052175</td>
      <td>-0.308546</td>
    </tr>
    <tr>
      <th>transaction_date</th>
      <td>-0.043490</td>
      <td>-0.025068</td>
      <td>-0.034520</td>
      <td>-0.034559</td>
      <td>-0.009964</td>
      <td>1.000000</td>
      <td>-0.415458</td>
      <td>0.014457</td>
      <td>-0.155364</td>
    </tr>
    <tr>
      <th>membership_expire_date</th>
      <td>-0.127438</td>
      <td>0.656398</td>
      <td>0.647513</td>
      <td>0.647361</td>
      <td>-0.184528</td>
      <td>-0.415458</td>
      <td>1.000000</td>
      <td>-0.009109</td>
      <td>0.286451</td>
    </tr>
    <tr>
      <th>is_cancel</th>
      <td>-0.020496</td>
      <td>0.005977</td>
      <td>0.017064</td>
      <td>0.016842</td>
      <td>0.052175</td>
      <td>0.014457</td>
      <td>-0.009109</td>
      <td>1.000000</td>
      <td>0.437393</td>
    </tr>
    <tr>
      <th>is_churn</th>
      <td>-0.203389</td>
      <td>0.455526</td>
      <td>0.447071</td>
      <td>0.444360</td>
      <td>-0.308546</td>
      <td>-0.155364</td>
      <td>0.286451</td>
      <td>0.437393</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



The user_logs dataframe contains interesting information related to each users intra day usage. I've also created some new features, such as:
 - counts_30: number of days the user used the service this month
 - counts_60: number of days the user used the service the month prior
 - num_unq_30: numbe of unique songs the user listened to this month
 - num_unq_30: numbe of unique songs the user listened to in the month prior
 - total_secs_30: total seconds of music listened to this month
 - total_secs_60: total seconds of music listened to in the month prior
 - days_since_last: how long it's been since the user last used the service (in days)
 


```python
# Explore user_logs

#convert dates to datetime obj
user_logs['date'] = pd.to_datetime(user_logs['date'], format = '%Y%m%d')

#print latest date
user_logs['date'].max()
```




    Timestamp('2017-03-31 00:00:00')




```python
# returns the max value of date and number of unique songs
# returns the sum of other variables
user_logs_v2 = user_logs.groupby('msno', as_index = False).agg({'date': 'max', 'num_25': 'sum', 'num_50': 'sum', 'num_75': 'sum',
                                 'num_985': 'sum', 'num_100': 'sum', 'num_unq': 'max', 'total_secs': 'sum'})

#change date to max_date
user_logs_v2.columns = ['msno', 'max_date', 'num_25', 'num_50', 'num_75', 'num_985', 'num_100',
       'num_unq', 'total_secs']

```


```python
#shift the dates back 30, 60 and 90 days
temp = user_logs.copy()
temp['max_date'] = user_logs_v2['max_date']
temp['last_30_days'] = temp['max_date'] - pd.Timedelta(days=30)
temp['last_60_days'] = temp['max_date'] - pd.Timedelta(days=60)
temp['last_90_days'] = temp['max_date'] - pd.Timedelta(days=90)

#group by msno to get total counts of days, total unique songs, and total seconds for 30/60/90 days
temp1 = temp[temp['date']>=temp['last_30_days']]
temp1_groupby = temp1.groupby(['msno']).agg({'msno': 'count', 'num_unq': 'sum', 'total_secs':'sum'})
temp1_groupby.columns = ['counts_30', 'num_unq_30', 'total_secs_30']
temp1_groupby.reset_index(inplace=True)

temp2 = temp[temp['date']>=temp['last_60_days']]
temp2_groupby = temp2.groupby(['msno']).agg({'msno': 'count', 'num_unq': 'sum', 'total_secs':'sum'})
temp2_groupby.columns = ['counts_60', 'num_unq_60', 'total_secs_60']
temp2_groupby.reset_index(inplace=True)


temp3 = temp[temp['date']>=temp['last_90_days']]
temp3_groupby = temp3.groupby(['msno']).agg({'msno': 'count', 'num_unq': 'sum', 'total_secs':'sum'})
temp3_groupby.columns = ['counts_90', 'num_unq_90', 'total_secs_90']
temp3_groupby.reset_index(inplace=True)


#merge back to user_logs_v2
user_logs_v2 = user_logs_v2.merge(temp1_groupby, on='msno', how='left')
user_logs_v2 = user_logs_v2.merge(temp2_groupby, on='msno', how='left')
user_logs_v2 = user_logs_v2.merge(temp3_groupby, on='msno', how='left')

```


```python
temp1_groupby.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>msno</th>
      <th>counts_30</th>
      <th>num_unq_30</th>
      <th>total_secs_30</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>+++IZseRRiQS9aaSkH6cMYU6bGDcxUieAi/tH67sC5s=</td>
      <td>3</td>
      <td>25</td>
      <td>4245.614</td>
    </tr>
    <tr>
      <th>1</th>
      <td>+++hVY1rZox/33YtvDgmKA2Frg/2qhkz12B9ylCvh8o=</td>
      <td>1</td>
      <td>5</td>
      <td>924.747</td>
    </tr>
    <tr>
      <th>2</th>
      <td>+++l/EXNMLTijfLBa8p2TUVVVp2aFGSuUI/h7mLmthw=</td>
      <td>1</td>
      <td>7</td>
      <td>1395.247</td>
    </tr>
    <tr>
      <th>3</th>
      <td>++0+IdHga8fCSioOVpU8K7y4Asw8AveIApVH2r9q9yY=</td>
      <td>2</td>
      <td>58</td>
      <td>10594.449</td>
    </tr>
    <tr>
      <th>4</th>
      <td>++0/NopttBsaAn6qHZA2AWWrDg7Me7UOMs1vsyo4tSI=</td>
      <td>2</td>
      <td>9</td>
      <td>1650.342</td>
    </tr>
  </tbody>
</table>
</div>




```python
#how long has it been since the user last used the service?
user_logs_v2['today'] = pd.to_datetime('2017-03-31 00:00:00')
#user_logs_v2['days_since_last'] = pd.Timedelta(user_logs_v2['today'] - user_logs_v2['max_date']).seconds
user_logs_v2['days_since_last'] = (user_logs_v2['today'] - user_logs_v2['max_date']).dt.days

```


```python
# to make these columns more useful, convert them to percentages
# calculate the percentage of number of songs played within certain period
user_logs_v2['percent_25'] = user_logs_v2['num_25']/(user_logs_v2['num_25']+user_logs_v2['num_50']+user_logs_v2['num_75']+user_logs_v2['num_985']+user_logs_v2['num_100'])
user_logs_v2['percent_50'] = user_logs_v2['num_50']/(user_logs_v2['num_25']+user_logs_v2['num_50']+user_logs_v2['num_75']+user_logs_v2['num_985']+user_logs_v2['num_100'])
user_logs_v2['percent_100'] = (user_logs_v2['num_985']+user_logs_v2['num_100'])/(user_logs_v2['num_25']+user_logs_v2['num_50']+user_logs_v2['num_75']+user_logs_v2['num_985']+user_logs_v2['num_100'])

# drop useless variables
user_logs_v3 = user_logs_v2.drop(columns = ['num_25', 'num_50', 'num_75', 'num_985', 'num_100'])
```


```python
#explore correlation with target variable
# user_logs_v3.merge(train, on='msno',how='inner').corr()
```

## Building the training data

Create the training data by combining all the data together into one big df. Next train the machine learning model using a variety of different classifer and ensemble models. 

I've also created one additional feauture:
- days_since_reg : total number of days the user has been a subscriber


```python
# merge between different tables for modelling purpose
dataset_train = train.merge(members, on = 'msno', how = 'left').merge(transactions_v2, on = 'msno', how = 'left').merge(user_logs_v3, on = 'msno', how = 'left')


```


```python
#create one more features


#convert to datetime objs
dataset_train['transaction_date'] = pd.to_datetime(dataset_train['transaction_date'], format = '%Y%m%d')
dataset_train['membership_expire_date'] = pd.to_datetime(dataset_train['membership_expire_date'], format = '%Y%m%d')

#length of usage in days (transaction_date is the registration date since groupby looked for the min value)
dataset_train['days_since_reg'] = (dataset_train['max_date'] - dataset_train['transaction_date']).dt.days

```


```python
#dataset_train.corr()
```

Next we will remove rows which are not useful in predicting churn, such as msno, bd, transaction_date, etc. 
The data set contains lot of missing values, so we will need to figure out a method to fill in the missing values with mean/mode.


```python
# remove useless rows AND
#remove gender and age since missing value or incorrect value is over 50%
dataset_train_v2 = dataset_train.drop(columns = ['msno', 'gender', 'bd', 'registration_init_time', 'transaction_date', 'membership_expire_date', 'max_date', 'today'])
dataset_train_v2.dtypes

# check the number of missing values in each column
dataset_train_v2.isna().sum()
```




    is_churn                   0
    city                  109993
    registered_via        109993
    payment_method_id      37382
    payment_plan_days      37382
    plan_list_price        37382
    actual_amount_paid     37382
    is_auto_renew          37382
    is_cancel              37382
    num_unq               216409
    total_secs            216409
    counts_30             514347
    num_unq_30            514347
    total_secs_30         514347
    counts_60             514347
    num_unq_60            514347
    total_secs_60         514347
    counts_90             514347
    num_unq_90            514347
    total_secs_90         514347
    days_since_last       216409
    percent_25            216409
    percent_50            216409
    percent_100           216409
    days_since_reg        245224
    dtype: int64




```python
# Handle missing value of part of numeric columns by using mode
def replacemode(i):
    dataset_train_v2[i] = dataset_train_v2[i].fillna(dataset_train_v2[i].value_counts().index[0])
    return 

replacemode('city')
replacemode('registered_via')
replacemode('payment_method_id')
replacemode('payment_plan_days')
replacemode('is_auto_renew')
replacemode('is_cancel')

# Handle missing value of part of numeric columns by using mean
from sklearn.impute import SimpleImputer
mean_imputer = SimpleImputer(missing_values = np.nan, strategy = 'mean')

def replacemean(i):
    dataset_train_v2[i] = mean_imputer.fit_transform(dataset_train_v2[[i]])
    return 

replacemean('plan_list_price')
replacemean('actual_amount_paid')
replacemean('num_unq')
replacemean('total_secs')
replacemean('percent_25')
replacemean('percent_50')
replacemean('percent_100')
replacemean('days_since_reg')
replacemean('days_since_last')
replacemean('counts_30')
replacemean('counts_60')
replacemean('counts_90')
replacemean('num_unq_30')
replacemean('num_unq_60')
replacemean('num_unq_90')
replacemean('total_secs_30')
replacemean('total_secs_60')
replacemean('total_secs_90')



# Handle outliers by using capping
def replaceoutlier(i):
    mean, std = np.mean(dataset_train_v2[i]), np.std(dataset_train_v2[i])
    cut_off = std*3
    lower, upper = mean - cut_off, mean + cut_off
    dataset_train_v2[i][dataset_train_v2[i] < lower] = lower
    dataset_train_v2[i][dataset_train_v2[i] > upper] = upper
    return

replaceoutlier('plan_list_price')
replaceoutlier('actual_amount_paid')
replaceoutlier('num_unq')
replaceoutlier('total_secs')
replaceoutlier('percent_25')
replaceoutlier('percent_50')
replaceoutlier('percent_100')
replaceoutlier('days_since_reg')
replaceoutlier('days_since_last')
replaceoutlier('counts_30')
replaceoutlier('counts_60')
replaceoutlier('counts_90')
replaceoutlier('num_unq_30')
replaceoutlier('num_unq_60')
replaceoutlier('num_unq_90')
replaceoutlier('total_secs_30')
replaceoutlier('total_secs_60')
replaceoutlier('total_secs_90')

```

In an effort to make the training time faster, I will be using feature scaling and dimension reduction to vectorize the data. 


```python
# Feature Scaling for modelling purpose by using both min-max-scaling
from sklearn.preprocessing import MinMaxScaler
X = dataset_train_v2.drop(columns = ['is_churn'])
Y = dataset_train_v2['is_churn']
nm_X = pd.DataFrame(MinMaxScaler().fit_transform(X))
nm_X.columns = X.columns.values
nm_X.index = X.index.values

```


```python
#pca reduces dimensionality, and only keeps most important vector
#cons: lose data and accuracy

# Dimension Reduction
from sklearn.decomposition import PCA
pca = PCA()
pca.fit_transform(nm_X)
np.cumsum(pca.explained_variance_ratio_)
mpca=PCA(n_components=10)
nm_X_v2 = mpca.fit_transform(nm_X)

# Split into train and test Set
from sklearn.model_selection import train_test_split
nm_X_train, nm_X_test, nm_Y_train, nm_Y_test = train_test_split(nm_X_v2, Y, test_size = 0.3, random_state = 0)
```


```python
# Fit training set into different algorithms

#to make things simple, only using logistic regression and xgb classifier

nm_models = []
# nm_models.append(('KNN', KNeighborsClassifier()))
nm_models.append(('LR', LogisticRegression()))
# nm_models.append(('LDA', LinearDiscriminantAnalysis()))
# nm_models.append(('QDA', QuadraticDiscriminantAnalysis()))
# nm_models.append(('CART', DecisionTreeClassifier()))
# nm_models.append(('NB', GaussianNB()))
# # nm_models.append(('Linear SVM', SVC(kernel = 'linear')))
# # nm_models.append(('Kernel SVM', SVC(kernel = 'rbf')))

ensembles = []
# ensembles.append(('BC', BaggingClassifier(base_estimator=LogisticRegression())))
# ensembles.append(('AB', AdaBoostClassifier(base_estimator=LogisticRegression())))
# ensembles.append(('GBM', GradientBoostingClassifier()))
# ensembles.append(('RF', RandomForestClassifier()))
# ensembles.append(('ET', ExtraTreesClassifier()))
ensembles.append(('XGB', XGBClassifier()))


#nm_models
from sklearn.model_selection import cross_val_score
results = []
names = []
for name, model in nm_models:
	nm_cv_results = cross_val_score(model, nm_X_train, nm_Y_train, cv=10, scoring='neg_log_loss', n_jobs = -1)
	results.append(nm_cv_results)
	names.append(name)
	msg = "%s: %f (%f)" % (name, nm_cv_results.mean(), nm_cv_results.std())
	print(msg)
    
#ensembles   
results2 = []
names2 = []
for name2, model2 in ensembles:
	en_cv_results = cross_val_score(model2, nm_X_train, nm_Y_train, cv=10, scoring='neg_log_loss', n_jobs = -1)
	results2.append(en_cv_results)
	names2.append(name2)
	msg2 = "%s: %f (%f)" % (name2, en_cv_results.mean(), en_cv_results.std())
	print(msg2)
```

    LR: -0.214069 (0.001141)
    XGB: -0.148262 (0.001632)


The best performing classifier model was the logistic regression model, and the best performing ensemble model was the XGB classifier (the rest have been commented out to speed up re-training).


```python
#evaluate feature importance

from numpy import loadtxt
from xgboost import XGBClassifier
from xgboost import plot_importance
from matplotlib import pyplot

# fit model to training data
model = XGBClassifier()
model.fit(nm_X_train, nm_Y_train)
# plot feature importance
plot_importance(model)
pyplot.show()
```

    [20:49:18] WARNING: /Users/travis/build/dmlc/xgboost/src/learner.cc:1061: Starting in XGBoost 1.3.0, the default evaluation metric used with the objective 'binary:logistic' was changed from 'error' to 'logloss'. Explicitly set eval_metric if you'd like to restore the old behavior.



    
![png](output_34_1.png)
    


According to the feature importance chart, "is_auto_renew" is the most important feature in determing wether a customer will churn, followed by the "payment_method_id" and "is_cancel". 
The top 10 are:
 - is_auto_renew
 - payment_method_id
 - is_cancel
 - num_unq
 - actual_amount_paid
 - registered_via
 - plan_list_price
 - payment_plan_days
 - city
 - total_secs



```python
# Apply Grid Search on XGBoost since it returns the best result on Cross Validation among all models
#find the optimal parameters for the model
from sklearn.model_selection import GridSearchCV
parameters = {"learning_rate": [0.1],
              "min_child_weight": [1],
              "max_depth": [9],
              "gamma": [0.1],
              "subsample": [0.8],
              "colsample_bytree": [0.8],
              "objective": ['binary:logistic']}
grid_search_XGB = GridSearchCV(estimator = XGBClassifier(), param_grid = parameters, scoring = "neg_log_loss", cv = 10, n_jobs = -1)
grid_result_XGB = grid_search_XGB.fit(nm_X_train, nm_Y_train)
print("Best: %f using %s" % (grid_result_XGB.best_score_, grid_result_XGB.best_params_)) 

# Evaluate tuned XGBoost model result on test dataset because it provides the best result
from sklearn.metrics import log_loss
nm_Y_predict = grid_result_XGB.predict_proba(nm_X_test)
logloss = log_loss(nm_Y_test, nm_Y_predict)  
print("Log Loss Score: %s" % (logloss))
```


```python
#save model so you don't have to retrain
import joblib
joblib.dump(grid_result_XGB.best_estimator_, 'trained-models/xgboost_model_v2.pkl')
```




    ['trained-models/xgboost_model_v2.pkl']




```python
#Do predictions
final=nm_X.copy()
final["is_churn"]=model.predict(nm_X_v2)
```


```python
#change column name to "pred_churn" since it's our predicted results
final.columns = ['city', 'registered_via', 'payment_method_id', 'payment_plan_days',
       'plan_list_price', 'actual_amount_paid', 'is_auto_renew', 'is_cancel',
       'num_unq', 'total_secs', 'counts_30', 'num_unq_30', 'total_secs_30',
       'counts_60', 'num_unq_60', 'total_secs_60', 'counts_90', 'num_unq_90',
       'total_secs_90', 'days_since_last', 'percent_25', 'percent_50',
       'percent_100', 'days_since_reg', 'pred_churn']
```


```python
#let's add the "is_churn" column (which contains the true churn values) back to the dataframe 
final['is_churn'] = dataset_train['is_churn']
```


```python
#confusion matrix
from sklearn.metrics import confusion_matrix
y_true = final['is_churn']
y_pred = final['pred_churn']
confusion_matrix(y_true, y_pred)


```




    array([[877477,   6153],
           [ 36325,  51005]])




```python
# outcome values order in sklearn
tp, fn, fp, tn = confusion_matrix(y_true,y_pred).reshape(-1)
print('Outcome values : \n', tp, fn, fp, tn)

print('\n')

# classification report for precision, recall f1-score and accuracy
from sklearn.metrics import classification_report
matrix = classification_report(y_true, y_pred)
print('Classification report : \n',matrix)
```

    Outcome values : 
     877477 6153 36325 51005
    
    
    Classification report : 
                   precision    recall  f1-score   support
    
               0       0.96      0.99      0.98    883630
               1       0.89      0.58      0.71     87330
    
        accuracy                           0.96    970960
       macro avg       0.93      0.79      0.84    970960
    weighted avg       0.95      0.96      0.95    970960
    



```python
#accuracy
acc = (tp + tn)/(tp + tn + fp + fn)
acc
```




    0.9562515448628162



So far, we see that model has a high accuracy score of 95% and a low log loss value. The model performed poorly in recall, specifically for predicting churned customers (is_churn = 1). The purpose of this project is to retarget customers that the model is predicting will churn, and try to keep them as customers before they cancel their subscription. Targeting customers who are incorrectly predicted to churn (i.e false positives) are okay; however, missing customers who are incorrectly predicted not to churn (false negatives) would be much worse. We see that the model has a much lower recall when predicting customers who will churn to customers who will not. The model was only able to predict 58% of the actual churned cases, meaning that a lot of customers who will churn will be missed. 

To improve the model further, we can try to remove non-important features and compare our scores. From there, we can also do more feature engineering and comapare once more. 

