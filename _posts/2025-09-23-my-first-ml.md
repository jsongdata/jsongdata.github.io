---
title: "My First ML Project"
date: 2025-09-23
categories: [Machine Learning]
tags: [machine learning]     # TAG names should always be lowercase
---

### The First Machine Learning Project for everyone

#### Background

This project is based on the first ML algorithm introduced in my favorite Korean machine learning book (ISBN: 9791158393229). The algorithm used here is the Decision Tree, which is known as one of the simplest ML algorithms. For this first project, I believe the book simply wanted to show how to import and use each module, rather than diving deep into the concepts. Therefore, I've added somewhat detailed information about ML assessment at the bottom of this post, drawing on the knowledge I learned during my B.S. in Data Analytics at The Ohio State University.

**Check Scikit Learn version**


```python
# View Scikit Learn version
import sklearn
print(sklearn.__version__)
```

    1.0.1


**Load needed modules**


```python
from sklearn.datasets import load_iris
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import train_test_split # This helps split data easy
```

**Load dataset**


```python
import pandas as pd

# Load iris data
iris = load_iris()

# iris.data includes the data itself without column names
iris_data = iris.data

# iris.target contains the labels (target values) of the iris dataset as a numpy array
iris_label = iris.target
print('iris target values:', iris_label)
print('iris target names:', iris.target_names)

# Convert the iris dataset into a DataFrame for better inspection
iris_df = pd.DataFrame(data=iris_data, columns=iris.feature_names)
iris_df['label'] = iris.target
iris_df.head(3)
```

    iris target values: [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
     0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
     1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 2
     2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2
     2 2]
    iris target names: ['setosa' 'versicolor' 'virginica']





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
      <th>sepal length (cm)</th>
      <th>sepal width (cm)</th>
      <th>petal length (cm)</th>
      <th>petal width (cm)</th>
      <th>label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5.1</td>
      <td>3.5</td>
      <td>1.4</td>
      <td>0.2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.9</td>
      <td>3.0</td>
      <td>1.4</td>
      <td>0.2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.7</td>
      <td>3.2</td>
      <td>1.3</td>
      <td>0.2</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



**Separate data into train(80%) and test(20%)**


```python
X_train, X_test, y_train, y_test = train_test_split(iris_data, iris_label, 
                                                    test_size=0.2, random_state=11)
```

**Train the model**


```python
# Create a DecisionTreeClassifier object
dt_clf = DecisionTreeClassifier(random_state=11)

# Train the model
dt_clf.fit(X_train, y_train)
```




    DecisionTreeClassifier(random_state=11)



**Predict**


```python
# Perform prediction on the test set with the trained DecisionTreeClassifier.
pred = dt_clf.predict(X_test)
pred
```






    array([2, 2, 1, 1, 2, 0, 1, 0, 0, 1, 1, 1, 1, 2, 2, 0, 2, 1, 2, 2, 1, 0,
           0, 1, 0, 0, 2, 1, 0, 1])



**Evaluate Accuracy Score**


```python
from sklearn.metrics import accuracy_score
print('Accuracy Score: {0:.4f}'.format(accuracy_score(y_test,pred)))
```

    Accuracy Score: 0.9333


An accuracy score of 0.9 (90%) is generally a high value, but that alone isn't enough to conclude that a model is good. Whether a model is truly effective depends on the type of problem you're trying to solve and the **characteristics of the data**. There are also many additional factors to consider, such as precision, recall, and the confusion matrix. However, since this project is meant for beginners and serves as a test for my first upload, I will conclude here.



