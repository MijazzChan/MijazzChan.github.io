---
title: Decision Tree Wine Classification
author: MijazzChan
date: 2021-05-11 03:41:29 +0800
categories: [Personal_Notes]
tags: [notes]
---

> Go to this [gist](https://gist.github.com/MijazzChan/00f39ed1f17e6026ab3b1be94fc8e636) for original files.


```python
# -*- coding: utf-8 -*-
# @Author: MijazzChan, 2017326603075
# @ => https://mijazz.icu
# Python Version == 3.8
import os
import pandas as pd
import numpy as np
from matplotlib import pyplot as plt
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.tree import DecisionTreeClassifier, export_text
from sklearn.utils import resample
from sklearn.metrics import confusion_matrix, classification_report
import warnings
%matplotlib inline
plt.rcParams['figure.dpi'] = 150
plt.rcParams['savefig.dpi'] = 150
sns.set(rc={"figure.dpi": 150, 'savefig.dpi': 150})
from jupyterthemes import jtplot
jtplot.style(theme='monokai', context='notebook', ticks=True, grid=False)
from IPython.core.display import HTML
HTML("""
<style>
.output_png {
    display: table-cell;
    text-align: center;
    vertical-align: middle;
}
</style>
""");
```

## Data Preprocessing

+ Reading from `csv`.
+ Replace `space( )` to `underscore(_)` in column names
+ check whether data containing nan/null.
+ check if data gets any duplicated entries.


```python
original_data = pd.read_csv('./winequality-red.csv', encoding='utf-8')
# Tweaks header. GET RID OF THOSE DAMN SPACE!
new_columns = list(map(lambda col: str(col).replace(' ', '_'), original_data.columns.to_list()))
original_data.columns = new_columns
original_data.head(6)
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
      <th>fixed_acidity</th>
      <th>volatile_acidity</th>
      <th>citric_acid</th>
      <th>residual_sugar</th>
      <th>chlorides</th>
      <th>free_sulfur_dioxide</th>
      <th>total_sulfur_dioxide</th>
      <th>density</th>
      <th>pH</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7.4</td>
      <td>0.70</td>
      <td>0.00</td>
      <td>1.9</td>
      <td>0.076</td>
      <td>11.0</td>
      <td>34.0</td>
      <td>0.9978</td>
      <td>3.51</td>
      <td>0.56</td>
      <td>9.4</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7.8</td>
      <td>0.88</td>
      <td>0.00</td>
      <td>2.6</td>
      <td>0.098</td>
      <td>25.0</td>
      <td>67.0</td>
      <td>0.9968</td>
      <td>3.20</td>
      <td>0.68</td>
      <td>9.8</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7.8</td>
      <td>0.76</td>
      <td>0.04</td>
      <td>2.3</td>
      <td>0.092</td>
      <td>15.0</td>
      <td>54.0</td>
      <td>0.9970</td>
      <td>3.26</td>
      <td>0.65</td>
      <td>9.8</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>11.2</td>
      <td>0.28</td>
      <td>0.56</td>
      <td>1.9</td>
      <td>0.075</td>
      <td>17.0</td>
      <td>60.0</td>
      <td>0.9980</td>
      <td>3.16</td>
      <td>0.58</td>
      <td>9.8</td>
      <td>6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7.4</td>
      <td>0.66</td>
      <td>0.00</td>
      <td>1.8</td>
      <td>0.075</td>
      <td>13.0</td>
      <td>40.0</td>
      <td>0.9978</td>
      <td>3.51</td>
      <td>0.56</td>
      <td>9.4</td>
      <td>5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>7.9</td>
      <td>0.60</td>
      <td>0.06</td>
      <td>1.6</td>
      <td>0.069</td>
      <td>15.0</td>
      <td>59.0</td>
      <td>0.9964</td>
      <td>3.30</td>
      <td>0.46</td>
      <td>9.4</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Check whether data got null/na mixed inside.
original_data.isna().sum()
```




    fixed_acidity           0
    volatile_acidity        0
    citric_acid             0
    residual_sugar          0
    chlorides               0
    free_sulfur_dioxide     0
    total_sulfur_dioxide    0
    density                 0
    pH                      0
    sulphates               0
    alcohol                 0
    quality                 0
    dtype: int64




```python
original_data.describe().T
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
      <th>count</th>
      <th>mean</th>
      <th>std</th>
      <th>min</th>
      <th>25%</th>
      <th>50%</th>
      <th>75%</th>
      <th>max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>fixed_acidity</th>
      <td>1359.0</td>
      <td>8.310596</td>
      <td>1.736990</td>
      <td>4.60000</td>
      <td>7.1000</td>
      <td>7.9000</td>
      <td>9.20000</td>
      <td>15.90000</td>
    </tr>
    <tr>
      <th>volatile_acidity</th>
      <td>1359.0</td>
      <td>0.529478</td>
      <td>0.183031</td>
      <td>0.12000</td>
      <td>0.3900</td>
      <td>0.5200</td>
      <td>0.64000</td>
      <td>1.58000</td>
    </tr>
    <tr>
      <th>citric_acid</th>
      <td>1359.0</td>
      <td>0.272333</td>
      <td>0.195537</td>
      <td>0.00000</td>
      <td>0.0900</td>
      <td>0.2600</td>
      <td>0.43000</td>
      <td>1.00000</td>
    </tr>
    <tr>
      <th>residual_sugar</th>
      <td>1359.0</td>
      <td>2.523400</td>
      <td>1.352314</td>
      <td>0.90000</td>
      <td>1.9000</td>
      <td>2.2000</td>
      <td>2.60000</td>
      <td>15.50000</td>
    </tr>
    <tr>
      <th>chlorides</th>
      <td>1359.0</td>
      <td>0.088124</td>
      <td>0.049377</td>
      <td>0.01200</td>
      <td>0.0700</td>
      <td>0.0790</td>
      <td>0.09100</td>
      <td>0.61100</td>
    </tr>
    <tr>
      <th>free_sulfur_dioxide</th>
      <td>1359.0</td>
      <td>15.893304</td>
      <td>10.447270</td>
      <td>1.00000</td>
      <td>7.0000</td>
      <td>14.0000</td>
      <td>21.00000</td>
      <td>72.00000</td>
    </tr>
    <tr>
      <th>total_sulfur_dioxide</th>
      <td>1359.0</td>
      <td>46.825975</td>
      <td>33.408946</td>
      <td>6.00000</td>
      <td>22.0000</td>
      <td>38.0000</td>
      <td>63.00000</td>
      <td>289.00000</td>
    </tr>
    <tr>
      <th>density</th>
      <td>1359.0</td>
      <td>0.996709</td>
      <td>0.001869</td>
      <td>0.99007</td>
      <td>0.9956</td>
      <td>0.9967</td>
      <td>0.99782</td>
      <td>1.00369</td>
    </tr>
    <tr>
      <th>pH</th>
      <td>1359.0</td>
      <td>3.309787</td>
      <td>0.155036</td>
      <td>2.74000</td>
      <td>3.2100</td>
      <td>3.3100</td>
      <td>3.40000</td>
      <td>4.01000</td>
    </tr>
    <tr>
      <th>sulphates</th>
      <td>1359.0</td>
      <td>0.658705</td>
      <td>0.170667</td>
      <td>0.33000</td>
      <td>0.5500</td>
      <td>0.6200</td>
      <td>0.73000</td>
      <td>2.00000</td>
    </tr>
    <tr>
      <th>alcohol</th>
      <td>1359.0</td>
      <td>10.432315</td>
      <td>1.082065</td>
      <td>8.40000</td>
      <td>9.5000</td>
      <td>10.2000</td>
      <td>11.10000</td>
      <td>14.90000</td>
    </tr>
    <tr>
      <th>quality</th>
      <td>1359.0</td>
      <td>5.623252</td>
      <td>0.823578</td>
      <td>3.00000</td>
      <td>5.0000</td>
      <td>6.0000</td>
      <td>6.00000</td>
      <td>8.00000</td>
    </tr>
  </tbody>
</table>
</div>




```python
original_data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1359 entries, 0 to 1358
    Data columns (total 12 columns):
     #   Column                Non-Null Count  Dtype  
    ---  ------                --------------  -----  
     0   fixed_acidity         1359 non-null   float64
     1   volatile_acidity      1359 non-null   float64
     2   citric_acid           1359 non-null   float64
     3   residual_sugar        1359 non-null   float64
     4   chlorides             1359 non-null   float64
     5   free_sulfur_dioxide   1359 non-null   float64
     6   total_sulfur_dioxide  1359 non-null   float64
     7   density               1359 non-null   float64
     8   pH                    1359 non-null   float64
     9   sulphates             1359 non-null   float64
     10  alcohol               1359 non-null   float64
     11  quality               1359 non-null   int64  
    dtypes: float64(11), int64(1)
    memory usage: 127.5 KB



```python
# Look for duplicates
original_data.duplicated().sum()
```




    0



### On quality(target) column

it seems that `quality` column only contains a few unique values. Go check that out shall we.


```python
original_data['quality'].unique().tolist()
```




    [5, 6, 7, 4, 8, 3]




```python
#original_data.groupby(['quality']).size().to_dict()
quality_plot_data = original_data.groupby(['quality']).size().reset_index()
quality_plot_data.columns = ['quality', 'count']
quality_plot_data.plot(kind='bar', x='quality', y='count', rot=0)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25ff8aa4df0>




![png](/assets/img/blog/20210511/output_9_1.png)


## Data visualization

### Box plots

Have a look at how each column data affects others.


```python
# tempX appended after each var is sort of a anti-corruption method? For me...
warnings.filterwarnings("ignore")
fig_temp1, ax_temp1 = plt.subplots(4, 3, figsize=(32, 24))
rolling_index = 0
columns_temp1 = list(original_data.columns)
columns_temp1.remove('quality')
for fig_row in range(4):
    for fig_col in range(3):
        sns.boxplot(x='quality', y=columns_temp1[rolling_index], data=original_data, ax=ax_temp1[fig_row][fig_col])
        rolling_index += 1
        if (rolling_index >= len(columns_temp1)):
            break
plt.show()
```


![png](/assets/img/blog/20210511/output_11_0.png)


### Heatmap on `corr()`


```python
# Need Square plot here temporaily
plt.figure(figsize=(16, 16))
sns.heatmap(original_data.corr(), annot=True, cmap='vlag')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25fff997a00>




![png](/assets/img/blog/20210511/output_13_1.png)


### Distributions(distplot)


```python
# Anti-variable-corrupt kicks in.
warnings.filterwarnings("ignore")
fig_temp2, ax_temp2 = plt.subplots(4, 3, figsize=(32, 24))
rolling_index = 0
columns_temp2 = list(original_data.columns)
columns_temp2.remove('quality')
for fig_row in range(4):
    for fig_col in range(3):
        sns.distplot(original_data[columns_temp2[rolling_index]], ax=ax_temp2[fig_row][fig_col])
        rolling_index += 1
        if (rolling_index >= len(columns_temp2)):
            break

```


![png](/assets/img/blog/20210511/output_15_0.png)


### Minor tweaks based on the visualization plot

#### Why?
It is noticeable that some data distributing in a skew way. No good for training.

> TODO: log trans? or sigmoid trans? MijazzChan @ 20210510220732 

Let's add some transformation shall we.

+ residual sugar
+ chlorides
+ free SO2
+ total SO2
+ sulphates
+ alcohol


```python
def trans_tweaks(col):
    return np.log(col[0])

tweaked_data = original_data.copy()
cols_need_tweaks = ['residual_sugar', 'chlorides', 'free_sulfur_dioxide', 'total_sulfur_dioxide', 'sulphates', 'alcohol']
for each_col in cols_need_tweaks:
    tweaked_data[each_col] = original_data[[each_col]].apply(trans_tweaks, axis=1)
# Tweaks completed.
```

#### Tweak result

distribution after transform.


```python
fig_temp3, ax_temp3 = plt.subplots(4, 3, figsize=(32, 24))
rolling_index = 0
columns_temp3 = list(tweaked_data.columns)
columns_temp3.remove('quality')
for fig_row in range(4):
    for fig_col in range(3):
        sns.distplot(tweaked_data[columns_temp3[rolling_index]], ax=ax_temp3[fig_row][fig_col])
        rolling_index += 1
        if (rolling_index >= len(columns_temp3)):
            break
```


![png](/assets/img/blog/20210511/output_19_0.png)


## Data Learning

### Only with `np.log` transform

Use sklearn build-in `DecisionTree` to make a approach.


```python
# use sklearn.model_selection.train_test_split to split train and test data.
X_temp1 = tweaked_data.drop(['quality'], axis=1)
Y_temp1 = tweaked_data['quality']
x_train, x_test, y_train, y_test = train_test_split(X_temp1, Y_temp1, random_state=99)
print(x_train.shape, x_test.shape, y_train.shape, y_test.shape)
print('which is {0} rows in training set, consisting of {1} features.'.format(x_train.shape[0], x_train.shape[1]))
print('Corespondingly, {} rows are included in testing set.'.format(x_test.shape[0]))
```

    (1019, 11) (340, 11) (1019,) (340,)
    which is 1019 rows in training set, consisting of 11 features.
    Corespondingly, 340 rows are included in testing set.



```python
# ID3 is entropy based DecisionTree.
entropy_decision_tree = DecisionTreeClassifier(criterion='entropy', max_depth=6)
# CART is gini based DecisionTree.
gini_decision_tree = DecisionTreeClassifier(criterion='gini', max_depth=6)

trees_apoch1 = [entropy_decision_tree, gini_decision_tree]
# Fit/Train
for tree in trees_apoch1:
    tree.fit(x_train, y_train)
# Predict
entropy_predition = entropy_decision_tree.predict(x_test)
gini_predition = gini_decision_tree.predict(x_test)

# score
entropy_score = accuracy_score(y_test, entropy_predition)
gini_score = accuracy_score(y_test, gini_predition)

# print the damn result
print('ID3 - Entropy Decision Tree (prediction score) ==> {}%'.format(np.round(entropy_score*100, decimals=3)))
print('CART - Gini Decision Tree (prediction score)   ==> {}%'.format(np.round(gini_score*100, decimals=3)))
```

    ID3 - Entropy Decision Tree (prediction score) ==> 58.235%
    CART - Gini Decision Tree (prediction score)   ==> 54.118%


> TODO: Introducing confusion matrix and classification report.

> MijazzChan@20210510-231909

> TODO-FINISHED AT 20210511-001102 BY MIJAZZCHAN



```python
print(' ID3 - Entropy Decision Tree \n    Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==>\n{}'
      .format(confusion_matrix(y_test, entropy_predition)))
print('    Classification Report 或 分类预测详细 ==>\n')
print(classification_report(y_test, entropy_predition))
print('*'*50 + '\n')
print(' CART - Gini Decision Tree \n   Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==> \n{}'
     .format(confusion_matrix(y_test, gini_predition)))
print('    Classification Report 或 分类预测详细 ==>\n')
print(classification_report(y_test, gini_predition))
```

     ID3 - Entropy Decision Tree 
        Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==>
    [[  0   1   2   0   0   0]
     [  0   3   8   0   0   0]
     [  0   0 109  28   2   0]
     [  0   3  56  67   7   0]
     [  0   1  10  22  19   0]
     [  0   0   0   1   1   0]]
        Classification Report 或 分类预测详细 ==>
    
                  precision    recall  f1-score   support
    
               3       0.00      0.00      0.00         3
               4       0.38      0.27      0.32        11
               5       0.59      0.78      0.67       139
               6       0.57      0.50      0.53       133
               7       0.66      0.37      0.47        52
               8       0.00      0.00      0.00         2
    
        accuracy                           0.58       340
       macro avg       0.36      0.32      0.33       340
    weighted avg       0.58      0.58      0.57       340
    
    **************************************************
    
     CART - Gini Decision Tree 
       Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==> 
    [[ 0  0  3  0  0  0]
     [ 1  0  7  3  0  0]
     [ 0  0 94 42  3  0]
     [ 0  0 48 74 10  1]
     [ 0  0  8 28 16  0]
     [ 0  0  0  1  1  0]]
        Classification Report 或 分类预测详细 ==>
    
                  precision    recall  f1-score   support
    
               3       0.00      0.00      0.00         3
               4       0.00      0.00      0.00        11
               5       0.59      0.68      0.63       139
               6       0.50      0.56      0.53       133
               7       0.53      0.31      0.39        52
               8       0.00      0.00      0.00         2
    
        accuracy                           0.54       340
       macro avg       0.27      0.26      0.26       340
    weighted avg       0.52      0.54      0.52       340


​    

### Train after grouped `quality`

It's worth noticing that this prediction outcome falls into 6 zones.

Including `quality` in [3, 4, 5, 6, 7, 8].

Perhaps a little grouping may help.

DEFINE {quality == 3 || quality == 4} AS 1 (LOW  quality)

DEFINE {quality == 5 || quality == 6} AS 2 (MID  quality)

DEFINE {quality == 7 || quality == 8} AS 3 (HIGH quality)

> Note that data prediction in this sector will only falls into [1, 2, 3] => [(3, 4), (5, 6), (7, 8)].


```python
def rate_transform(col):
    if col[0] < 5:
        return 1
    elif col[0] < 7:
        return 2
    return 3

tweaked_data['rate'] = tweaked_data[['quality']].apply(rate_transform, axis=1)
rate_plot_data = tweaked_data.groupby(['rate']).size().reset_index()
rate_plot_data.columns = ['rate', 'count']
rate_plot_data.plot(kind='bar', x='rate', y='count', rot=0)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x25fff26d700>




![png](/assets/img/blog/20210511/output_26_1.png)


This time, prediction will falls into 3 zones. As define above, RATE => [1, 2, 3]
Try re-fit.
This time, drop `quality` and `rate`. `rate` will be y.


```python
X_temp2 = tweaked_data.drop(['quality', 'rate'], axis=1)
Y_temp2 = tweaked_data['rate']
x_train, x_test, y_train, y_test = train_test_split(X_temp2, Y_temp2, random_state=99)
print(x_train.shape, x_test.shape, y_train.shape, y_test.shape)
print('which is {0} rows in training set, consisting of {1} features.'.format(x_train.shape[0], x_train.shape[1]))
print('Corespondingly, {} rows are included in testing set.\n'.format(x_test.shape[0]))
entropy_decision_tree = DecisionTreeClassifier(criterion='entropy', max_depth=5)
gini_decision_tree = DecisionTreeClassifier(criterion='gini', max_depth=5)

trees_apoch2 = [entropy_decision_tree, gini_decision_tree]
for tree in trees_apoch2:
    tree.fit(x_train, y_train)
# Predict
entropy_predition = entropy_decision_tree.predict(x_test)
gini_predition = gini_decision_tree.predict(x_test)
# score
entropy_score = accuracy_score(y_test, entropy_predition)
gini_score = accuracy_score(y_test, gini_predition)

# print the result again.
print('ID3 - Entropy Decision Tree (prediction score) ==> {}%'.format(np.round(entropy_score*100, decimals=3)))
print('CART - Gini Decision Tree (prediction score)   ==> {}%'.format(np.round(gini_score*100, decimals=3)))
```

    (1019, 11) (340, 11) (1019,) (340,)
    which is 1019 rows in training set, consisting of 11 features.
    Corespondingly, 340 rows are included in testing set.
    
    ID3 - Entropy Decision Tree (prediction score) ==> 82.059%
    CART - Gini Decision Tree (prediction score)   ==> 78.824%


> TODO: Introducing confusion matrix and classification report here.

> MijazzChan@20210510-232102

> TODO-FINISHED AT 20210511-002652 BY MijazzChan



```python
print(' ID3 - Entropy Decision Tree \n    Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==>\n{}'
      .format(confusion_matrix(y_test, entropy_predition)))
print('    Classification Report 或 分类预测详细 ==>\n')
print(classification_report(y_test, entropy_predition))
print('*'*50 + '\n')
print(' CART - Gini Decision Tree \n   Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==> \n{}'
     .format(confusion_matrix(y_test, gini_predition)))
print('    Classification Report 或 分类预测详细 ==>\n')
print(classification_report(y_test, gini_predition))
```

     ID3 - Entropy Decision Tree 
        Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==>
    [[  0  14   0]
     [  1 260  11]
     [  0  35  19]]
        Classification Report 或 分类预测详细 ==>
    
                  precision    recall  f1-score   support
    
               1       0.00      0.00      0.00        14
               2       0.84      0.96      0.90       272
               3       0.63      0.35      0.45        54
    
        accuracy                           0.82       340
       macro avg       0.49      0.44      0.45       340
    weighted avg       0.77      0.82      0.79       340
    
    **************************************************
    
     CART - Gini Decision Tree 
       Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==> 
    [[  2  12   0]
     [  3 250  19]
     [  0  38  16]]
        Classification Report 或 分类预测详细 ==>
    
                  precision    recall  f1-score   support
    
               1       0.40      0.14      0.21        14
               2       0.83      0.92      0.87       272
               3       0.46      0.30      0.36        54
    
        accuracy                           0.79       340
       macro avg       0.56      0.45      0.48       340
    weighted avg       0.76      0.79      0.77       340


​    

### Learning on tweaked data 

#### How to resample

Y-data seems a little bit inbalanced?
Okay then, add some resample might work.

Going back to `quality` column, which contains 6 different values.

Hence drop the `rate` column we added before.


```python
# Drop the rate data we added before.
tweaked_data.drop(['rate'], inplace=True, axis=1)
df_3 = tweaked_data[tweaked_data['quality'] == 3]
df_4 = tweaked_data[tweaked_data['quality'] == 4]
df_5 = tweaked_data[tweaked_data['quality'] == 5]
df_6 = tweaked_data[tweaked_data['quality'] == 6]
df_7 = tweaked_data[tweaked_data['quality'] == 7]
df_8 = tweaked_data[tweaked_data['quality'] == 8]
i = 3
for each_df in [df_3, df_4, df_5, df_6, df_7, df_8]:
    print('quality == {0} has {1} entries'.format(i, each_df.shape[0]))
    i += 1
```

    quality == 3 has 10 entries
    quality == 4 has 53 entries
    quality == 5 has 577 entries
    quality == 6 has 535 entries
    quality == 7 has 167 entries
    quality == 8 has 17 entries


It's quite obvious that 3, 4, 7, 8 are minorities. 5, 6 are majorities.

Hence we up-sample 3, 4, 6, 7, 8 to 550 entries.
down-sample 5 to 550 entries


```python
df_3_upsampled = resample(df_3, replace=True, n_samples=550, random_state=9) 
df_4_upsampled = resample(df_4, replace=True, n_samples=550, random_state=9) 
df_7_upsampled = resample(df_7, replace=True, n_samples=550, random_state=9) 
df_8_upsampled = resample(df_8, replace=True, n_samples=550, random_state=9) 
df_5_downsampled = df_5.sample(n=550).reset_index(drop=True)
df_6_upsampled = resample(df_6, replace=True, n_samples=550, random_state=9) 
```

After Re-sample(upsample & downsample). Value count looks like this.


```python
resampled_dfs = [df_3_upsampled, df_4_upsampled, df_5_downsampled, df_6_upsampled, df_7_upsampled, df_8_upsampled]
resampled_data = pd.concat(resampled_dfs, axis=0)
i = 3
for each_df in resampled_dfs:
    print('quality == {0} has {1} entries'.format(i, each_df.shape[0]))
    i += 1
```

    quality == 3 has 550 entries
    quality == 4 has 550 entries
    quality == 5 has 550 entries
    quality == 6 has 550 entries
    quality == 7 has 550 entries
    quality == 8 has 550 entries


#### Get rid of some "unrelated" data

Before data is sent to training. Minor tweak is advised here.

> TODO: pre-train data process. MijazzChan@20210511-010817
> TODO-FINISH: MijazzChan@20210511-032652

Try drop some unrelated columns/features?


```python
resampled_data.corr()['quality']
```




    fixed_acidity           0.125105
    volatile_acidity       -0.600034
    citric_acid             0.384477
    residual_sugar          0.009648
    chlorides              -0.377277
    free_sulfur_dioxide     0.081144
    total_sulfur_dioxide    0.057946
    density                -0.321477
    pH                     -0.267204
    sulphates               0.490420
    alcohol                 0.597137
    quality                 1.000000
    Name: quality, dtype: float64



little note here:

> `corr()` values is `皮尔逊积矩相关系数` 或`correlation coefficient`. It stays within [-1, 1].

abs(corr()) 越逼近1, 即相关度越高.

取abs后, 再看这些因素与`quality`的相关性.

After mapping with abs, we can review how much these columns are co-related with `quality`.


```python
def map_abs(col):
    return np.abs(col[0])

corr_df = resampled_data.corr()['quality'].reset_index()

corr_df.columns = ['factors', 'correlation']
# quality is always related to quality, which means quality.corr == 1
# Hence the drop.
corr_df = corr_df[corr_df['factors'] != 'quality']
# Mapping with abs function
corr_df['correlation(abs)'] = corr_df[['correlation']].apply(map_abs, axis=1)
corr_df.sort_values(by='correlation(abs)', ascending=False, inplace=True)
corr_df.drop(['correlation'], inplace=True, axis=1)
print('TOP 5 quality-related factors\n', corr_df.head(5))
sns.barplot(x='correlation(abs)', y='factors', data=corr_df, orient='h', palette='flare')
```

    TOP 5 quality-related factors
                  factors  correlation(abs)
    1   volatile_acidity          0.600034
    10           alcohol          0.597137
    9          sulphates          0.490420
    2        citric_acid          0.384477
    4          chlorides          0.377277





    <matplotlib.axes._subplots.AxesSubplot at 0x25fff35f040>




![png](/assets/img/blog/20210511/output_40_2.png)


Note that only following factors is worth putting into training. They are 

+ volatile acidity
+ alcohol
+ sulphates
+ citric acid
+ chlorides
+ density
+ pH
+ free sulfur dioxide
+ fixed acidity
+ total sulfur dioxide

Dropping factors as follows: 

+ residual sugar


```python
factors_to_drop = ['residual_sugar']

simplified_data = resampled_data[resampled_data.columns[~resampled_data.columns.isin(factors_to_drop)]]
simplified_data.head(6)
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
      <th>fixed_acidity</th>
      <th>volatile_acidity</th>
      <th>citric_acid</th>
      <th>chlorides</th>
      <th>free_sulfur_dioxide</th>
      <th>total_sulfur_dioxide</th>
      <th>density</th>
      <th>pH</th>
      <th>sulphates</th>
      <th>alcohol</th>
      <th>quality</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1106</th>
      <td>7.6</td>
      <td>1.580</td>
      <td>0.00</td>
      <td>-1.987774</td>
      <td>1.609438</td>
      <td>2.197225</td>
      <td>0.99476</td>
      <td>3.50</td>
      <td>-0.916291</td>
      <td>2.388763</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1165</th>
      <td>6.8</td>
      <td>0.815</td>
      <td>0.00</td>
      <td>-1.320507</td>
      <td>2.772589</td>
      <td>3.367296</td>
      <td>0.99471</td>
      <td>3.32</td>
      <td>-0.673345</td>
      <td>2.282382</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1253</th>
      <td>7.1</td>
      <td>0.875</td>
      <td>0.05</td>
      <td>-2.501036</td>
      <td>1.098612</td>
      <td>2.639057</td>
      <td>0.99808</td>
      <td>3.40</td>
      <td>-0.653926</td>
      <td>2.322388</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1165</th>
      <td>6.8</td>
      <td>0.815</td>
      <td>0.00</td>
      <td>-1.320507</td>
      <td>2.772589</td>
      <td>3.367296</td>
      <td>0.99471</td>
      <td>3.32</td>
      <td>-0.673345</td>
      <td>2.282382</td>
      <td>3</td>
    </tr>
    <tr>
      <th>450</th>
      <td>10.4</td>
      <td>0.610</td>
      <td>0.49</td>
      <td>-1.609438</td>
      <td>1.609438</td>
      <td>2.772589</td>
      <td>0.99940</td>
      <td>3.16</td>
      <td>-0.462035</td>
      <td>2.128232</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1165</th>
      <td>6.8</td>
      <td>0.815</td>
      <td>0.00</td>
      <td>-1.320507</td>
      <td>2.772589</td>
      <td>3.367296</td>
      <td>0.99471</td>
      <td>3.32</td>
      <td>-0.673345</td>
      <td>2.282382</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>



Put the simplified df into training. See how it performs.


```python
X_temp3 = simplified_data.drop(['quality'], axis=1)
Y_temp3 = simplified_data['quality']
x_train, x_test, y_train, y_test = train_test_split(X_temp3, Y_temp3, random_state=99, test_size=0.3)

# Note that you don't use resampled data to test!!!!!!
# Drop duplicated row you added when resample.

# test_df = pd.concat([x_test, y_test], axis=1)
# print('test data containing duplicated entries(added during resample) => {}, drop them.'.format(test_df.duplicated().sum()))
# test_df.drop_duplicates(inplace=True)
# x_test = test_df.drop(['quality'], axis=1)
# y_test = test_df['quality']
print(x_train.shape, x_test.shape, y_train.shape, y_test.shape)
print('which is {0} rows in training set, consisting of {1} features.'.format(x_train.shape[0], x_train.shape[1]))
print('Corespondingly, {} rows are included in testing set.\n'.format(x_test.shape[0]))
entropy_decision_tree = DecisionTreeClassifier(criterion='entropy', splitter='random')
gini_decision_tree = DecisionTreeClassifier(criterion='gini', splitter='random')

trees_apoch3 = [entropy_decision_tree, gini_decision_tree]
for tree in trees_apoch3:
    tree.fit(x_train, y_train)
# Predict
entropy_predition = entropy_decision_tree.predict(x_test)
gini_predition = gini_decision_tree.predict(x_test)
# score
entropy_score = accuracy_score(y_test, entropy_predition)
gini_score = accuracy_score(y_test, gini_predition)

# print the result.
print('ID3 - Entropy Decision Tree (prediction score) ==> {}%'.format(np.round(entropy_score*100, decimals=3)))
print('CART - Gini Decision Tree (prediction score)   ==> {}%'.format(np.round(gini_score*100, decimals=3)))
```

    (2310, 10) (990, 10) (2310,) (990,)
    which is 2310 rows in training set, consisting of 10 features.
    Corespondingly, 990 rows are included in testing set.
    
    ID3 - Entropy Decision Tree (prediction score) ==> 89.293%
    CART - Gini Decision Tree (prediction score)   ==> 89.697%



```python
print(' ID3 - Entropy Decision Tree \n    Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==>\n{}'
      .format(confusion_matrix(y_test, entropy_predition)))
print('    Classification Report 或 分类预测详细 ==>\n')
print(classification_report(y_test, entropy_predition))
print('*'*50 + '\n')
print(' CART - Gini Decision Tree \n   Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==> \n{}'
     .format(confusion_matrix(y_test, gini_predition)))
print('    Classification Report 或 分类预测详细 ==>\n')
print(classification_report(y_test, gini_predition))
```

     ID3 - Entropy Decision Tree 
        Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==>
    [[180   0   0   0   0   0]
     [  0 172   0   0   0   0]
     [  0  10  88  32   9   0]
     [  0   5  31 119   9   2]
     [  0   0   3   3 149   2]
     [  0   0   0   0   0 176]]
        Classification Report 或 分类预测详细 ==>
    
                  precision    recall  f1-score   support
    
               3       1.00      1.00      1.00       180
               4       0.92      1.00      0.96       172
               5       0.72      0.63      0.67       139
               6       0.77      0.72      0.74       166
               7       0.89      0.95      0.92       157
               8       0.98      1.00      0.99       176
    
        accuracy                           0.89       990
       macro avg       0.88      0.88      0.88       990
    weighted avg       0.89      0.89      0.89       990
    
    **************************************************
    
     CART - Gini Decision Tree 
       Confusion Matrix 或 判断矩阵 或 真假阴阳性矩阵 ==> 
    [[180   0   0   0   0   0]
     [  0 172   0   0   0   0]
     [  3  10  84  31   9   2]
     [  0   4  22 125  11   4]
     [  0   0   0   4 151   2]
     [  0   0   0   0   0 176]]
        Classification Report 或 分类预测详细 ==>
    
                  precision    recall  f1-score   support
    
               3       0.98      1.00      0.99       180
               4       0.92      1.00      0.96       172
               5       0.79      0.60      0.69       139
               6       0.78      0.75      0.77       166
               7       0.88      0.96      0.92       157
               8       0.96      1.00      0.98       176
    
        accuracy                           0.90       990
       macro avg       0.89      0.89      0.88       990
    weighted avg       0.89      0.90      0.89       990


​    

The outcome is quite satisfying given that the predition this time falls into 6 zones.

这次预测的结果已经比较让人满意了, 因为预测值并非像先前加入rate一样, 只预测3个结果(LOW MID HIGH).
本次的预测结果是直接落入准确的`[3,4,5,6,7,8]`里的.

make a simple text visualization on the `DecisionTree`



```python
print(export_text(entropy_decision_tree, feature_names=x_train.columns.to_list()))
```

    |--- sulphates <= -0.38
    |   |--- total_sulfur_dioxide <= 3.96
    |   |   |--- volatile_acidity <= 0.93
    |   |   |   |--- chlorides <= -1.91
    |   |   |   |   |--- pH <= 3.15
    |   |   |   |   |   |--- volatile_acidity <= 0.37
    |   |   |   |   |   |   |--- free_sulfur_dioxide <= 1.73
    |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 2.58
    |   |   |   |   |   |   |   |   |--- pH <= 3.12
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- pH >  3.12
    |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  2.58
    |   |   |   |   |   |   |   |   |--- class: 8
    |   |   |   |   |   |   |--- free_sulfur_dioxide >  1.73
    |   |   |   |   |   |   |   |--- volatile_acidity <= 0.30
    |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.30
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.30
    |   |   |   |   |   |   |   |   |   |--- volatile_acidity <= 0.24
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.30
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.30
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- volatile_acidity >  0.24
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- volatile_acidity >  0.30
    |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.55
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.55
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- volatile_acidity >  0.37
    |   |   |   |   |   |   |--- alcohol <= 2.38
    |   |   |   |   |   |   |   |--- pH <= 3.08
    |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.65
    |   |   |   |   |   |   |   |   |   |--- pH <= 3.06
    |   |   |   |   |   |   |   |   |   |   |--- chlorides <= -2.81
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- chlorides >  -2.81
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- pH >  3.06
    |   |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.65
    |   |   |   |   |   |   |   |   |   |--- chlorides <= -2.63
    |   |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |   |   |--- chlorides >  -2.63
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- pH >  3.08
    |   |   |   |   |   |   |   |   |--- sulphates <= -0.55
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 2.93
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  2.93
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- sulphates >  -0.55
    |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.39
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.39
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.57
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.57
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- alcohol >  2.38
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.23
    |   |   |   |   |   |   |   |   |--- chlorides <= -2.51
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- chlorides >  -2.51
    |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.23
    |   |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |--- pH >  3.15
    |   |   |   |   |   |--- sulphates <= -0.70
    |   |   |   |   |   |   |--- chlorides <= -2.89
    |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 2.31
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  2.31
    |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- chlorides >  -2.89
    |   |   |   |   |   |   |   |--- chlorides <= -2.55
    |   |   |   |   |   |   |   |   |--- sulphates <= -0.98
    |   |   |   |   |   |   |   |   |   |--- pH <= 3.31
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- pH >  3.31
    |   |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |   |--- sulphates >  -0.98
    |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.34
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.09
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.09
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 4
    |   |   |   |   |   |   |   |   |   |--- alcohol >  2.34
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.32
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.32
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |--- chlorides >  -2.55
    |   |   |   |   |   |   |   |   |--- chlorides <= -2.17
    |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.28
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.02
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.02
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 4
    |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.28
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- chlorides >  -2.17
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |--- sulphates >  -0.70
    |   |   |   |   |   |   |--- citric_acid <= 0.64
    |   |   |   |   |   |   |   |--- volatile_acidity <= 0.70
    |   |   |   |   |   |   |   |   |--- alcohol <= 2.35
    |   |   |   |   |   |   |   |   |   |--- fixed_acidity <= 6.07
    |   |   |   |   |   |   |   |   |   |   |--- volatile_acidity <= 0.56
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- volatile_acidity >  0.56
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |   |   |--- fixed_acidity >  6.07
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.16
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 11
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.16
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 10
    |   |   |   |   |   |   |   |   |--- alcohol >  2.35
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.81
    |   |   |   |   |   |   |   |   |   |   |--- volatile_acidity <= 0.38
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 8
    |   |   |   |   |   |   |   |   |   |   |--- volatile_acidity >  0.38
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 9
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.81
    |   |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.44
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 5
    |   |   |   |   |   |   |   |   |   |   |--- alcohol >  2.44
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |--- volatile_acidity >  0.70
    |   |   |   |   |   |   |   |   |--- alcohol <= 2.38
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.22
    |   |   |   |   |   |   |   |   |   |   |--- fixed_acidity <= 8.41
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 5
    |   |   |   |   |   |   |   |   |   |   |--- fixed_acidity >  8.41
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.22
    |   |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.28
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- alcohol >  2.28
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 4
    |   |   |   |   |   |   |   |   |--- alcohol >  2.38
    |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.42
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.18
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.18
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- alcohol >  2.42
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.74
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.74
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |--- citric_acid >  0.64
    |   |   |   |   |   |   |   |--- pH <= 3.23
    |   |   |   |   |   |   |   |   |--- fixed_acidity <= 11.71
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.07
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.07
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- fixed_acidity >  11.71
    |   |   |   |   |   |   |   |   |   |--- sulphates <= -0.40
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |--- sulphates >  -0.40
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |--- pH >  3.23
    |   |   |   |   |   |   |   |   |--- fixed_acidity <= 10.27
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- fixed_acidity >  10.27
    |   |   |   |   |   |   |   |   |   |--- class: 3
    |   |   |   |--- chlorides >  -1.91
    |   |   |   |   |--- free_sulfur_dioxide <= 1.98
    |   |   |   |   |   |--- alcohol <= 2.24
    |   |   |   |   |   |   |--- pH <= 3.30
    |   |   |   |   |   |   |   |--- class: 3
    |   |   |   |   |   |   |--- pH >  3.30
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |--- alcohol >  2.24
    |   |   |   |   |   |   |--- chlorides <= -1.65
    |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |--- chlorides >  -1.65
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |--- free_sulfur_dioxide >  1.98
    |   |   |   |   |   |--- alcohol <= 2.28
    |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- alcohol >  2.28
    |   |   |   |   |   |   |--- class: 3
    |   |   |--- volatile_acidity >  0.93
    |   |   |   |--- density <= 1.00
    |   |   |   |   |--- volatile_acidity <= 1.57
    |   |   |   |   |   |--- volatile_acidity <= 1.25
    |   |   |   |   |   |   |--- density <= 0.99
    |   |   |   |   |   |   |   |--- volatile_acidity <= 0.98
    |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |--- volatile_acidity >  0.98
    |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |--- density >  0.99
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 1.83
    |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  1.83
    |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.05
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.05
    |   |   |   |   |   |   |   |   |   |--- fixed_acidity <= 7.82
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- fixed_acidity >  7.82
    |   |   |   |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |--- volatile_acidity >  1.25
    |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |--- volatile_acidity >  1.57
    |   |   |   |   |   |--- class: 3
    |   |   |   |--- density >  1.00
    |   |   |   |   |--- total_sulfur_dioxide <= 3.02
    |   |   |   |   |   |--- class: 3
    |   |   |   |   |--- total_sulfur_dioxide >  3.02
    |   |   |   |   |   |--- free_sulfur_dioxide <= 2.97
    |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.58
    |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.58
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |--- free_sulfur_dioxide >  2.97
    |   |   |   |   |   |   |--- class: 3
    |   |--- total_sulfur_dioxide >  3.96
    |   |   |--- free_sulfur_dioxide <= 2.55
    |   |   |   |--- alcohol <= 2.33
    |   |   |   |   |--- sulphates <= -0.51
    |   |   |   |   |   |--- citric_acid <= 0.33
    |   |   |   |   |   |   |--- fixed_acidity <= 8.67
    |   |   |   |   |   |   |   |--- pH <= 3.29
    |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.47
    |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.47
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- pH >  3.29
    |   |   |   |   |   |   |   |   |--- sulphates <= -0.52
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- sulphates >  -0.52
    |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |--- fixed_acidity >  8.67
    |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- citric_acid >  0.33
    |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |--- sulphates >  -0.51
    |   |   |   |   |   |--- class: 5
    |   |   |   |--- alcohol >  2.33
    |   |   |   |   |--- total_sulfur_dioxide <= 4.31
    |   |   |   |   |   |--- alcohol <= 2.53
    |   |   |   |   |   |   |--- density <= 0.99
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- density >  0.99
    |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- alcohol >  2.53
    |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |--- total_sulfur_dioxide >  4.31
    |   |   |   |   |   |--- class: 4
    |   |   |--- free_sulfur_dioxide >  2.55
    |   |   |   |--- volatile_acidity <= 0.28
    |   |   |   |   |--- chlorides <= -2.90
    |   |   |   |   |   |--- citric_acid <= 0.24
    |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- citric_acid >  0.24
    |   |   |   |   |   |   |--- alcohol <= 2.52
    |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |--- alcohol >  2.52
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |--- chlorides >  -2.90
    |   |   |   |   |   |--- pH <= 3.34
    |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |--- class: 8
    |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |--- pH >  3.34
    |   |   |   |   |   |   |--- class: 5
    |   |   |   |--- volatile_acidity >  0.28
    |   |   |   |   |--- alcohol <= 2.44
    |   |   |   |   |   |--- alcohol <= 2.30
    |   |   |   |   |   |   |--- citric_acid <= 0.45
    |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |--- sulphates <= -0.57
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 4.69
    |   |   |   |   |   |   |   |   |   |   |--- fixed_acidity <= 7.64
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 6
    |   |   |   |   |   |   |   |   |   |   |--- fixed_acidity >  7.64
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  4.69
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- sulphates >  -0.57
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 4.62
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.02
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.02
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 8
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  4.62
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |--- citric_acid >  0.45
    |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- alcohol >  2.30
    |   |   |   |   |   |   |--- pH <= 3.74
    |   |   |   |   |   |   |   |--- volatile_acidity <= 0.57
    |   |   |   |   |   |   |   |   |--- chlorides <= -2.58
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 4.23
    |   |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.32
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |   |--- alcohol >  2.32
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  4.23
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 4.28
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  4.28
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- chlorides >  -2.58
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 4.46
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.23
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.23
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 4
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  4.46
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- volatile_acidity >  0.57
    |   |   |   |   |   |   |   |   |--- citric_acid <= 0.23
    |   |   |   |   |   |   |   |   |   |--- sulphates <= -0.51
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- sulphates >  -0.51
    |   |   |   |   |   |   |   |   |   |   |--- chlorides <= -2.59
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |   |--- chlorides >  -2.59
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- citric_acid >  0.23
    |   |   |   |   |   |   |   |   |   |--- volatile_acidity <= 1.01
    |   |   |   |   |   |   |   |   |   |   |--- sulphates <= -0.55
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |   |   |   |--- sulphates >  -0.55
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |--- volatile_acidity >  1.01
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.25
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.25
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |--- pH >  3.74
    |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |--- alcohol >  2.44
    |   |   |   |   |   |--- density <= 0.99
    |   |   |   |   |   |   |--- fixed_acidity <= 5.13
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- fixed_acidity >  5.13
    |   |   |   |   |   |   |   |--- alcohol <= 2.52
    |   |   |   |   |   |   |   |   |--- alcohol <= 2.48
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- alcohol >  2.48
    |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- alcohol >  2.52
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- density >  0.99
    |   |   |   |   |   |   |--- class: 6
    |--- sulphates >  -0.38
    |   |--- chlorides <= -2.17
    |   |   |--- alcohol <= 2.51
    |   |   |   |--- volatile_acidity <= 0.45
    |   |   |   |   |--- free_sulfur_dioxide <= 2.22
    |   |   |   |   |   |--- volatile_acidity <= 0.28
    |   |   |   |   |   |   |--- chlorides <= -2.34
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 1.91
    |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  1.91
    |   |   |   |   |   |   |   |   |--- sulphates <= -0.34
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- sulphates >  -0.34
    |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |--- chlorides >  -2.34
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |--- volatile_acidity >  0.28
    |   |   |   |   |   |   |--- alcohol <= 2.31
    |   |   |   |   |   |   |   |--- alcohol <= 2.27
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- alcohol >  2.27
    |   |   |   |   |   |   |   |   |--- class: 8
    |   |   |   |   |   |   |--- alcohol >  2.31
    |   |   |   |   |   |   |   |--- alcohol <= 2.43
    |   |   |   |   |   |   |   |   |--- volatile_acidity <= 0.34
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.10
    |   |   |   |   |   |   |   |   |   |   |--- sulphates <= -0.18
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |   |--- sulphates >  -0.18
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.10
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |--- volatile_acidity >  0.34
    |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.52
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 1.92
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  1.92
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.52
    |   |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.38
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- alcohol >  2.38
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |--- alcohol >  2.43
    |   |   |   |   |   |   |   |   |--- alcohol <= 2.47
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 1.52
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  1.52
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 1.83
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  1.83
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |--- alcohol >  2.47
    |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.51
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |--- alcohol >  2.51
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |--- free_sulfur_dioxide >  2.22
    |   |   |   |   |   |--- total_sulfur_dioxide <= 3.84
    |   |   |   |   |   |   |--- alcohol <= 2.41
    |   |   |   |   |   |   |   |--- chlorides <= -2.34
    |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.37
    |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.33
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.19
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.19
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |--- alcohol >  2.33
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.37
    |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.38
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.76
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.76
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- alcohol >  2.38
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- chlorides >  -2.34
    |   |   |   |   |   |   |   |   |--- fixed_acidity <= 11.70
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- fixed_acidity >  11.70
    |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |--- alcohol >  2.41
    |   |   |   |   |   |   |   |--- sulphates <= -0.27
    |   |   |   |   |   |   |   |   |--- alcohol <= 2.45
    |   |   |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- alcohol >  2.45
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.58
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.58
    |   |   |   |   |   |   |   |   |   |   |--- pH <= 3.26
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- pH >  3.26
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- sulphates >  -0.27
    |   |   |   |   |   |   |   |   |--- pH <= 3.17
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- pH >  3.17
    |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.53
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.53
    |   |   |   |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |--- total_sulfur_dioxide >  3.84
    |   |   |   |   |   |   |--- alcohol <= 2.39
    |   |   |   |   |   |   |   |--- alcohol <= 2.27
    |   |   |   |   |   |   |   |   |--- sulphates <= -0.34
    |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |   |--- sulphates >  -0.34
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 3.28
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  3.28
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- alcohol >  2.27
    |   |   |   |   |   |   |   |   |--- fixed_acidity <= 7.71
    |   |   |   |   |   |   |   |   |   |--- pH <= 3.40
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- pH >  3.40
    |   |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.29
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- alcohol >  2.29
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- fixed_acidity >  7.71
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 4.05
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.90
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.90
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  4.05
    |   |   |   |   |   |   |   |   |   |   |--- volatile_acidity <= 0.35
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |   |--- volatile_acidity >  0.35
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |--- alcohol >  2.39
    |   |   |   |   |   |   |   |--- sulphates <= -0.14
    |   |   |   |   |   |   |   |   |--- pH <= 3.18
    |   |   |   |   |   |   |   |   |   |--- fixed_acidity <= 8.50
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- fixed_acidity >  8.50
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |--- pH >  3.18
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 4.01
    |   |   |   |   |   |   |   |   |   |   |--- pH <= 3.36
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- pH >  3.36
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  4.01
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid <= 0.50
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- citric_acid >  0.50
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- sulphates >  -0.14
    |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |--- volatile_acidity >  0.45
    |   |   |   |   |--- alcohol <= 2.29
    |   |   |   |   |   |--- sulphates <= -0.33
    |   |   |   |   |   |   |--- chlorides <= -2.29
    |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.83
    |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.83
    |   |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- chlorides >  -2.29
    |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- sulphates >  -0.33
    |   |   |   |   |   |   |--- sulphates <= 0.02
    |   |   |   |   |   |   |   |--- citric_acid <= 0.04
    |   |   |   |   |   |   |   |   |--- volatile_acidity <= 0.85
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 1.81
    |   |   |   |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  1.81
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- volatile_acidity >  0.85
    |   |   |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |   |   |   |--- citric_acid >  0.04
    |   |   |   |   |   |   |   |   |--- fixed_acidity <= 8.19
    |   |   |   |   |   |   |   |   |   |--- pH <= 3.39
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- pH >  3.39
    |   |   |   |   |   |   |   |   |   |   |--- pH <= 3.49
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |   |--- pH >  3.49
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- fixed_acidity >  8.19
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |--- sulphates >  0.02
    |   |   |   |   |   |   |   |--- class: 4
    |   |   |   |   |--- alcohol >  2.29
    |   |   |   |   |   |--- alcohol <= 2.40
    |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.14
    |   |   |   |   |   |   |   |   |--- fixed_acidity <= 9.53
    |   |   |   |   |   |   |   |   |   |--- pH <= 3.41
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- pH >  3.41
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.00
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- fixed_acidity >  9.53
    |   |   |   |   |   |   |   |   |   |--- volatile_acidity <= 0.56
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |--- volatile_acidity >  0.56
    |   |   |   |   |   |   |   |   |   |   |--- fixed_acidity <= 12.25
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |   |--- fixed_acidity >  12.25
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.14
    |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 2.42
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  2.42
    |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.38
    |   |   |   |   |   |   |   |   |   |   |--- fixed_acidity <= 8.42
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- fixed_acidity >  8.42
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 3
    |   |   |   |   |   |   |   |   |   |--- alcohol >  2.38
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.60
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.60
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |--- alcohol >  2.40
    |   |   |   |   |   |   |--- alcohol <= 2.45
    |   |   |   |   |   |   |   |--- fixed_acidity <= 15.15
    |   |   |   |   |   |   |   |   |--- chlorides <= -2.77
    |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.41
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- alcohol >  2.41
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 3.17
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  3.17
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- chlorides >  -2.77
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |--- fixed_acidity >  15.15
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |--- alcohol >  2.45
    |   |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |   |--- volatile_acidity <= 0.58
    |   |   |   |   |   |   |   |   |   |--- sulphates <= -0.08
    |   |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |   |   |--- sulphates >  -0.08
    |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- volatile_acidity >  0.58
    |   |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |--- alcohol >  2.51
    |   |   |   |--- density <= 1.00
    |   |   |   |   |--- volatile_acidity <= 0.47
    |   |   |   |   |   |--- volatile_acidity <= 0.39
    |   |   |   |   |   |   |--- citric_acid <= 0.41
    |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |--- citric_acid >  0.41
    |   |   |   |   |   |   |   |--- pH <= 3.11
    |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |--- pH >  3.11
    |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |--- volatile_acidity >  0.39
    |   |   |   |   |   |   |--- citric_acid <= 0.16
    |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |--- citric_acid >  0.16
    |   |   |   |   |   |   |   |--- sulphates <= -0.29
    |   |   |   |   |   |   |   |   |--- chlorides <= -2.84
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |--- chlorides >  -2.84
    |   |   |   |   |   |   |   |   |   |--- class: 8
    |   |   |   |   |   |   |   |--- sulphates >  -0.29
    |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |--- volatile_acidity >  0.47
    |   |   |   |   |   |--- sulphates <= -0.17
    |   |   |   |   |   |   |--- total_sulfur_dioxide <= 3.81
    |   |   |   |   |   |   |   |--- fixed_acidity <= 7.19
    |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- fixed_acidity >  7.19
    |   |   |   |   |   |   |   |   |--- class: 8
    |   |   |   |   |   |   |--- total_sulfur_dioxide >  3.81
    |   |   |   |   |   |   |   |--- pH <= 3.40
    |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |--- pH >  3.40
    |   |   |   |   |   |   |   |   |--- class: 8
    |   |   |   |   |   |--- sulphates >  -0.17
    |   |   |   |   |   |   |--- free_sulfur_dioxide <= 1.93
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- free_sulfur_dioxide >  1.93
    |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |--- density >  1.00
    |   |   |   |   |--- chlorides <= -2.38
    |   |   |   |   |   |--- pH <= 3.20
    |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |--- pH >  3.20
    |   |   |   |   |   |   |--- class: 8
    |   |   |   |   |--- chlorides >  -2.38
    |   |   |   |   |   |--- class: 5
    |   |--- chlorides >  -2.17
    |   |   |--- chlorides <= -0.57
    |   |   |   |--- pH <= 3.28
    |   |   |   |   |--- total_sulfur_dioxide <= 2.69
    |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |--- citric_acid <= 0.42
    |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- citric_acid >  0.42
    |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |--- total_sulfur_dioxide >  2.69
    |   |   |   |   |   |--- alcohol <= 2.30
    |   |   |   |   |   |   |--- alcohol <= 2.22
    |   |   |   |   |   |   |   |--- chlorides <= -2.08
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- chlorides >  -2.08
    |   |   |   |   |   |   |   |   |--- sulphates <= 0.19
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide <= 4.80
    |   |   |   |   |   |   |   |   |   |   |--- alcohol <= 2.21
    |   |   |   |   |   |   |   |   |   |   |   |--- truncated branch of depth 2
    |   |   |   |   |   |   |   |   |   |   |--- alcohol >  2.21
    |   |   |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |   |   |   |--- total_sulfur_dioxide >  4.80
    |   |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- sulphates >  0.19
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |--- alcohol >  2.22
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide <= 3.13
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- free_sulfur_dioxide >  3.13
    |   |   |   |   |   |   |   |   |--- sulphates <= 0.12
    |   |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |   |--- sulphates >  0.12
    |   |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |--- alcohol >  2.30
    |   |   |   |   |   |   |--- alcohol <= 2.43
    |   |   |   |   |   |   |   |--- chlorides <= -1.94
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |   |--- chlorides >  -1.94
    |   |   |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |   |   |--- alcohol >  2.43
    |   |   |   |   |   |   |   |--- fixed_acidity <= 12.47
    |   |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |   |   |--- fixed_acidity >  12.47
    |   |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |--- pH >  3.28
    |   |   |   |   |--- fixed_acidity <= 8.41
    |   |   |   |   |   |--- sulphates <= -0.30
    |   |   |   |   |   |   |--- fixed_acidity <= 7.02
    |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |--- fixed_acidity >  7.02
    |   |   |   |   |   |   |   |--- class: 7
    |   |   |   |   |   |--- sulphates >  -0.30
    |   |   |   |   |   |   |--- class: 6
    |   |   |   |   |--- fixed_acidity >  8.41
    |   |   |   |   |   |--- citric_acid <= 0.43
    |   |   |   |   |   |   |--- density <= 1.00
    |   |   |   |   |   |   |   |--- class: 3
    |   |   |   |   |   |   |--- density >  1.00
    |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |--- citric_acid >  0.43
    |   |   |   |   |   |   |--- pH <= 3.36
    |   |   |   |   |   |   |   |--- class: 5
    |   |   |   |   |   |   |--- pH >  3.36
    |   |   |   |   |   |   |   |--- class: 4
    |   |   |--- chlorides >  -0.57
    |   |   |   |--- class: 4


​    
