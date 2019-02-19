
# Class Imbalance Problems

## Introduction


We've talked about precision, recall, ROC curves and AUC as metrics for evaluating the performance of classifiers. With this, we've seen how measuring the performance of classification algorithms is substantially different from that of regression. For example, we briefly discussed a scenario where only 2 in 1000 cases were labelled 'positive'. In such drastically imbalanced cases, even a naive classifier that simply always predicts a 'negative' label would be 99.8% accurate. Moreove, such scenarios are relatively common in areas such as medical conditions or credit card fraud. As such, there has been a lot of work and research regarding class imbalance problems and methods for tuning classification algorithms to better fit these scenarios.


## Objectives

You will be able to:
* Use modified sampling techniques to address class imbalance problems
    * Weighting (Class weighting) 
    * Generating new data (with similar distribution) - SMOTE
* Understand the complications of class imbalance problems
    * Screws up accuracy score 
        * If 2 classes are unbalanced, there will be a high accuracy score, because it's based on probability
            * Overfitting
    * SMOTE vs Class Weighting? Which one?
        * Weighting may add a ton of FP
        * SMOTE - there's no guarantee that all of the false data fits the population
* Describe scenarios that involve unbalanced classes
    * Diagnosing rare medical condition
    * Credit card fraud
    * Chatbots to understand language (rarely questions)
    * Ad serving

# Outline
* Go through this lab
* Discussing weighting samples by label
* Balancing samples using artificial data

## Class Weight

One initial option for dealing with class imbalance problems is to weight the two classes. By default the class weights for Logistic Regression in scikit-learn is none, meaning that both classes will be given equal importance in tuning the model. Alternatively, you can pass 'balanced' in order to assign weights that are inversely proportionally to that class's frequency. The final option is to explicitly pass weights to each class using a dictionary of the form {class_label: weight}.  

Questions:
What do weights do?

cause your model to train on each point by evaluating the weight associated. 


```python
# Classes : Weights
# [0, 1, 0, ..., 0, 1] counts 0: 9950, 1: 50
# multiply a label by the other label's proportions
# 0 : 0.005
# 1 : 0.995
```


```python
# An high accuracy model would just always predict 0
# Recall, Low f1, low precision
```

First, here's the documentation to take a further look:

```
class_weight : dict or 'balanced', default: None
    Weights associated with classes in the form ``{class_label: weight}``.
    If not given, all classes are supposed to have weight one.

    The "balanced" mode uses the values of y to automatically adjust
    weights inversely proportional to class frequencies in the input data
    as ``n_samples / (n_classes * np.bincount(y))``.

    Note that these weights will be multiplied with sample_weight (passed
    through the fit method) if sample_weight is specified.

    .. versionadded:: 0.17
       *class_weight='balanced'*
```

To investigate this, we'll load a dataset on Mobile App Downloads. The data represents information regarding users to a website and whether or not they downloaded the app for their phone. As you can guess, most vistors don't download the app making it an imbalanced dataset as we had been discussing.

First, we load in the dataset and some applicable packages.


```python
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score, roc_curve, auc
import pandas as pd


#Load the data
df = pd.read_csv('mobile_app_attribution.csv')

#Data Preview
df.head()
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
      <th>ip</th>
      <th>app</th>
      <th>device</th>
      <th>os</th>
      <th>channel</th>
      <th>click_time</th>
      <th>attributed_time</th>
      <th>is_attributed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>87540</td>
      <td>12</td>
      <td>1</td>
      <td>13</td>
      <td>497</td>
      <td>2017-11-07 09:30:38</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>105560</td>
      <td>25</td>
      <td>1</td>
      <td>17</td>
      <td>259</td>
      <td>2017-11-07 13:40:27</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>101424</td>
      <td>12</td>
      <td>1</td>
      <td>19</td>
      <td>212</td>
      <td>2017-11-07 18:05:24</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>94584</td>
      <td>13</td>
      <td>1</td>
      <td>13</td>
      <td>477</td>
      <td>2017-11-07 04:58:08</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>68413</td>
      <td>12</td>
      <td>1</td>
      <td>1</td>
      <td>178</td>
      <td>2017-11-09 09:00:09</td>
      <td>NaN</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.is_attributed.value_counts()
```




    0    99773
    1      227
    Name: is_attributed, dtype: int64



Then we'll define X and y and investigate the level of class imbalance.


```python
#Define appropriate X and y
X = pd.get_dummies(df[df.columns[:5]])
y = df.is_attributed
# Split the data into train and test sets.
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=0)

#Preview Level of Imbalance in Dataset
print(y.value_counts())
print(y.value_counts(normalize=True))
```

    0    99773
    1      227
    Name: is_attributed, dtype: int64
    0    0.99773
    1    0.00227
    Name: is_attributed, dtype: float64



```python
print(y_train.value_counts())
print(y_test.value_counts())
```

    0    74841
    1      159
    Name: is_attributed, dtype: int64
    0    24932
    1       68
    Name: is_attributed, dtype: int64


As you can see, over 99% of the data is the negative case. With that, let's compare a few models with varying class weights.

## Comparing Models with Varying Class Weights 


```python
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_style('darkgrid')
%matplotlib inline
```


```python
# Now let's compare a few different regularization performances on the dataset:
weights = [None, 'balanced', {1:2, 0:1}, {1:10, 0:1}, {1:100, 0:1}, {1:2000, 0:1}]
names = ['None', 'Balanced', '2 to 1', '10 to 1', '100 to 1', '2000 to 1']
colors = sns.color_palette("Set2")

plt.figure(figsize=(10,8))

for n, weight in enumerate(weights):
    #Fit a model
    logreg = LogisticRegression(fit_intercept = False, C = 1e12, class_weight=weight) #Starter code
    model_log = logreg.fit(X_train, y_train)
    print(model_log) #Preview model params

    #Predict
    y_hat_test = logreg.predict(X_test)

    y_score = logreg.fit(X_train, y_train).decision_function(X_test)

    fpr, tpr, thresholds = roc_curve(y_test, y_score)
    
    print('AUC for {}: {}'.format(names[n], auc(fpr, tpr)))
    lw = 2
    plt.plot(fpr, tpr, color=colors[n],
             lw=lw, label='ROC curve {}'.format(names[n]))
plt.plot([0, 1], [0, 1], color='navy', lw=lw, linestyle='--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])

plt.yticks([i/20.0 for i in range(21)])
plt.xticks([i/20.0 for i in range(21)])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Receiver operating characteristic (ROC) Curve')
plt.legend(loc="lower right")
plt.show()
```

    LogisticRegression(C=1000000000000.0, class_weight=None, dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for None: 0.5193473306216438
    LogisticRegression(C=1000000000000.0, class_weight='balanced', dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for Balanced: 0.8320938835986824
    LogisticRegression(C=1000000000000.0, class_weight={1: 2, 0: 1}, dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for 2 to 1: 0.5600792980436198
    LogisticRegression(C=1000000000000.0, class_weight={1: 10, 0: 1}, dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for 10 to 1: 0.6414730419682714
    LogisticRegression(C=1000000000000.0, class_weight={1: 100, 0: 1}, dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for 100 to 1: 0.7512740536612527
    LogisticRegression(C=1000000000000.0, class_weight={1: 2000, 0: 1},
              dual=False, fit_intercept=False, intercept_scaling=1,
              max_iter=100, multi_class='ovr', n_jobs=1, penalty='l2',
              random_state=None, solver='liblinear', tol=0.0001, verbose=0,
              warm_start=False)
    AUC for 2000 to 1: 0.8669439699512085



![png](index_files/index_16_1.png)


### Interpreting a ROC Curve
* x axis - fp rate
* y axis - tp rate
* yellow curve is best because
    * because it optimizes tp and fp rate
    * tp should be as high as we can get it
    * fp should be as low as we can get it
    * usually occurs at the critical point of the curve

As you can see, class weight can have a significant impact! In this case, typically the heavier that we weight the positive case, the better our classifier appears to be performing.

### Oversampling and Undersampling
Another technique that can be used is oversampling or undersampling. This can help address class imbalance problems when one category is far more prevelant then the other. This is a common case that occurs in medicine, image classification or fraud detection. In many of these scenarios, class imbalance can cause difficulties for the learning algorithm. After all, simply predicting the majority class could yield 99%+ accuracy if the rare class occurs <1% of the time. Due to this, sampling techniques such as oversampling the minority class or undersampling the majority class can help by producing a synthetic dataset that the learning algorithm is trained on. With this, it is important to still maintain a test set from the original dataset in order to accurately judge the accuracy of the algorithm overall.

While these initial modifications will improve the performance of classification algorithms on imbalanced datasets, a more advance technique known as SMOTE has produced even better results in practice.

#### SMOTE
SMOTE stands for Synthetic Minority Oversampling. Here, rather then simply oversampling the minority class with replacement (which simply adds duplicate cases to the dataset), the algorithm generates new sample data by creating 'synthetic' examples that are combinations of the closest minority class cases.   
   
   Implementing this technique is very easy using the **imblearn** package:


```python
from imblearn.over_sampling import SMOTE, ADASYN
```


```python
print(y.value_counts()) #Previous original class distribution
X_resampled, y_resampled = SMOTE().fit_sample(X, y) 
print(pd.Series(y_resampled).value_counts()) #Preview synthetic sample class distribution
```

    0    99773
    1      227
    Name: is_attributed, dtype: int64
    1    99773
    0    99773
    dtype: int64



```python
X_train, X_test, y_train, y_test = train_test_split(X_resampled, y_resampled, random_state=0)
```


```python
# Now let's compare a few different regularization performances on the dataset:
weights = [None, 'balanced', {1:2, 0:1}, {1:10, 0:1}, {1:100, 0:1}, {1:1000, 0:1}]
names = ['None', 'Balanced', '2 to 1', '10 to 1', '100 to 1', '1000 to 1']
colors = sns.color_palette("Set2")

plt.figure(figsize=(10,8))

for n, weight in enumerate(weights):
    #Fit a model
    logreg = LogisticRegression(fit_intercept = False, C = 1e12, class_weight=weight) #Starter code
    model_log = logreg.fit(X_train, y_train)
    print(model_log) #Preview model params

    #Predict
    y_hat_test = logreg.predict(X_test)

    y_score = logreg.fit(X_train, y_train).decision_function(X_test)

    fpr, tpr, thresholds = roc_curve(y_test, y_score)
    
    print('AUC for {}: {}'.format(names[n], auc(fpr, tpr)))
    lw = 2
    plt.plot(fpr, tpr, color=colors[n],
             lw=lw, label='ROC curve {}'.format(names[n]))
plt.plot([0, 1], [0, 1], color='navy', lw=lw, linestyle='--')
plt.xlim([0.0, 1.0])
plt.ylim([0.0, 1.05])

plt.yticks([i/20.0 for i in range(21)])
plt.xticks([i/20.0 for i in range(21)])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('Receiver operating characteristic (ROC) Curve')
plt.legend(loc="lower right")
plt.show()
```

    LogisticRegression(C=1000000000000.0, class_weight=None, dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for None: 0.850270382286993
    LogisticRegression(C=1000000000000.0, class_weight='balanced', dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for Balanced: 0.8478359001648685
    LogisticRegression(C=1000000000000.0, class_weight={1: 2, 0: 1}, dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for 2 to 1: 0.8608682717492752
    LogisticRegression(C=1000000000000.0, class_weight={1: 10, 0: 1}, dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for 10 to 1: 0.716443047560376
    LogisticRegression(C=1000000000000.0, class_weight={1: 100, 0: 1}, dual=False,
              fit_intercept=False, intercept_scaling=1, max_iter=100,
              multi_class='ovr', n_jobs=1, penalty='l2', random_state=None,
              solver='liblinear', tol=0.0001, verbose=0, warm_start=False)
    AUC for 100 to 1: 0.6530895282975107
    LogisticRegression(C=1000000000000.0, class_weight={1: 1000, 0: 1},
              dual=False, fit_intercept=False, intercept_scaling=1,
              max_iter=100, multi_class='ovr', n_jobs=1, penalty='l2',
              random_state=None, solver='liblinear', tol=0.0001, verbose=0,
              warm_start=False)
    AUC for 1000 to 1: 0.7122381175889384



![png](index_files/index_23_1.png)


Hopefully this should make sense; after synthetically resampling our data, we no longer need to lean on penalized class weights in order to improve our model tuning. Since SMOTE recreated our dataset to have a balanced number of positive and negative cases, aggressive weighting schemas such as 10:1, 100:1 or 1000:1 drastically impact our model performance; the data is effectively no longer class imbalanced, so creating the class weights effectively reintroduces the original problem. Overall, our SMOTE unweighted model appears to be the current top performer. In practice, it is up to you the modeler, to make this and other choices when comparing models. For example, you may also wish to tune other parameters in your model such as how to perform regularization.   

As a review, recall that regularization are penalties to a more straightforward problem of minimizing error between our model and its outputs. The most two common regularizations are the l1 lasso and l2 ridge penalties. These add additional complexity to the loss function. In scikit-learn, these can be specified when initializing your regression object as in:
```logreg = LogisticRegression(penalty='l1')```

The default is to use an l2 penalty, so unless you specified otherwise, that's what you've been using.   

In addition to simply specifying how to regularize the model, you can also specify the amount of regularization. This is controlled through the c parameter wh is class imbalance. For example, here is the ROC curve of various regularization values with no corrections for class imbalance:

<img src="roc_regularization.png">

As you can see, all of these models perform poorly regardless of the amout of regularization. 

## Summary
In this lesson we investigated various tuning parameters for our model, as well as dealing with class imbalance as a whole. In the upcoming lab, you'll have a chance to try and adjust these parameters yourself in order to optimize a model for predicting credit fraud.
