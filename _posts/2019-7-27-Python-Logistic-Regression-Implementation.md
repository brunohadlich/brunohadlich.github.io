---
layout: post
title: Implementing Logistic Regression in Python
---

This post has the intention of being a consultation base for those who need a Logistic Regression implementation that has been previously
tested against a reliable framework. Below is the code and if you have a good knowledge of python you can maybe understand how the
algorithm works by reading the code but this is not the purpose of this post, if you want to first understand the steps that compound this
algorithm I really recommend you to take [Andrew NG](https://www.coursera.org/instructor/andrewng)
[Machine Learning](https://www.coursera.org/learn/machine-learning) classes. The code presented below was developed by me, I based its
development on Andrew's Machine Learning course which primary programming language is Octave and decided to re-implement the algorithm in
python. When starting to read the code you may think that I am using libraries like pandas and sickit learn but that is only for data
cleaning and later comparison between a known framework like scikit learn and my implementation. In main scope there are two function
calls that better explain this, <strong><em>manual_solution()</strong></em> and <strong><em>scikit_learn_solution()</strong></em>, the
first one is completely written by me and the second one uses scikit learn, both of them use
[titanic passengers dataset](https://www.kaggle.com/lcqueiroz/titanic-passengers-dataset/data), even though you will find three csv files
only train.csv was used and then splitted in training set and test set. Before executing the code please make sure you are using python3 as
I used Python 3.7.3 during development and do not forget to install pandas and scikit learn with pip,
<strong>pip3 install pandas scikit-learn</strong>, after executing the code with <strong>python3 logistic_regression.py</strong> you will
see an output similar to:

```console
bruno@bruno-laptop:~/git/Programming-Studies/machine_learning/logistic_regression$ python3 logistic_regression.py

Processing manual solution:

              precision    recall  f1-score   support

           0       0.82      0.91      0.86       163
           1       0.83      0.68      0.75       104

    accuracy                           0.82       267
   macro avg       0.82      0.80      0.80       267
weighted avg       0.82      0.82      0.82       267

Accuracy: 0.8202247191011236

manual_solution took 77.73170065879822s to finish.

Processing scikit learn solution:

/home/bruno/.local/lib/python3.7/site-packages/sklearn/linear_model/logistic.py:432: FutureWarning: Default solver will be changed to 'lbfgs' in 0.22. Specify a solver to silence this warning.
  FutureWarning)
              precision    recall  f1-score   support

           0       0.81      0.93      0.86       163
           1       0.85      0.65      0.74       104

    accuracy                           0.82       267
   macro avg       0.83      0.79      0.80       267
weighted avg       0.82      0.82      0.81       267

Accuracy: 0.8202247191011236

scikit_learn_solution took 0.0959620475769043s to finish.

bruno@bruno-laptop:~/git/Programming-Studies/machine_learning/logistic_regression$
```

As you can see manual solution took 77 seconds against scikit learn 0.09 seconds, approximately 818 times longer, fortunately the
accuracy of both solutions was the same 0.8202247191011236. I was surprised when realized the exact same accuracy for both solutions but
that was achieved only after tuning the parameters of the function <strong><em>gradient_descent()</strong></em> using a limit of 10000
iterations, alpha equals to 0.01 and a threshold of convergence of 0.000001 between iterations. I expect this post to be useful, below is
the code and thank you for your time.

```python3
import pandas as pd
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report, accuracy_score
from math import sqrt, exp, log
from time import sleep, time


def sigmoid(x):
    return 1 / (1 + exp(-x))


def hypothesis(theta, x):
    return sigmoid(sum([theta[i] * x[i] for i in range(len(theta))]))


def cost_function(theta, X, Y):
    result = 0
    for x, y in zip(X, Y):
        h = hypothesis(theta, x)
        result += y * log(h) + (1 - y) * log(1 - h)

    return -(1 / len(X)) * result


def gradient_descent(x, y, max_iterations, alpha, convergence_threshold):
    iterations = 0
    convergence = 999999
    m = len(x)
    n = len(x[0])
    theta = [0] * n
    last_cost = cost_function(theta, x, y)
    history_cost_iteration = []
    while convergence > convergence_threshold and iterations < max_iterations:
        for j in range(n):
            result = 0
            for i in range(m):
                result += (hypothesis(theta, x[i]) - y[i]) * x[i][j]
            theta[j] -= (alpha / m) * result
        cost = cost_function(theta, x, y)
        convergence = abs(last_cost - cost)
        last_cost = cost
        iterations += 1
        history_cost_iteration.append(cost)
    return (theta, history_cost_iteration)


def mean(x):
    m = len(x)
    n = len(x[0])
    x_feature_mean = [0] * n
    for i in range(n):
        x_feature_mean[i] = sum([x[k][i] for k in range(m)]) / m
    return x_feature_mean


def standard_deviation(x):
    m = len(x)
    n = len(x[0])
    x_feature_mean = mean(x)
    x_feature_standard_deviation = [0] * n
    for i in range(n):
        result = 0
        for k in range(m):
            result += pow(x[k][i] - x_feature_mean[i], 2)
        x_feature_standard_deviation[i] = sqrt(1 / (m - 1) * result)
    return x_feature_standard_deviation


def feature_normalize(x):
    m = len(x)
    n = len(x[0])
    x_feature_mean = mean(x)
    x_feature_standard_deviation = standard_deviation(x)
    for i in range(n):
        for k in range(m):
            x[k][i] -= x_feature_mean[i]
            x[k][i] /= x_feature_standard_deviation[i]
    return (x, x_feature_mean, x_feature_standard_deviation)


def predict(x_test, x_feature_mean, x_feature_sdv, theta):
    for i, x in enumerate(x_test):
        result = []
        for k in range(len(x)):
            result.append((x[k] - x_feature_mean[k]) / x_feature_sdv[k])
        x_test[i] = result
    x_test = [[1] + x_test[i] for i in range(len(x_test))]
    m = len(x_test)
    predictions = [0] * m
    i = 0
    for el in x_test:
        predictions[i] = 1 if hypothesis(theta, el) >= 0.5 else 0
        i = i + 1
    return predictions


def inpute_age(cols):
    Age = cols[0]
    Pclass = cols[1]

    if pd.isnull(Age):

        if Pclass == 1:
            return 37
        elif Pclass == 2:
            return 29
        else:
            return 24
    else:
        return Age


def load_format_dataset():
    train = pd.read_csv("titanic_problem/train.csv")
    train["Age"] = train[["Age", "Pclass"]].apply(inpute_age, axis=1)
    train.drop("Cabin", axis=1, inplace=True)
    train.dropna(inplace=True)
    sex = pd.get_dummies(train["Sex"], drop_first=True)
    embark = pd.get_dummies(train["Embarked"], drop_first=True)
    train.drop(["Sex", "Embarked", "Name", "Ticket"], axis=1, inplace=True)
    train = pd.concat([train, sex, embark], axis=1)
    return train_test_split(
        train.drop("Survived", axis=1),
        train["Survived"],
        test_size=0.30,
        random_state=101,
    )


def print_accuracy(y_test, predictions):
    print(classification_report(y_test, predictions))
    print("Accuracy:", accuracy_score(y_test, predictions))


def scikit_learn_solution():
    X_train, X_test, y_train, y_test =  load_format_dataset()
    logmodel = LogisticRegression()
    logmodel.fit(X_train, y_train)
    predictions = logmodel.predict(X_test)
    print_accuracy(y_test, predictions)


def manual_solution():
    x_train, x_test, y_train, y_test = load_format_dataset()
    x_train = x_train.values.tolist()
    y_train = y_train.values.tolist()
    x_test = x_test.values.tolist()
    y_test = y_test.values.tolist()
    x_train, x_feature_mean, x_feature_sdv = feature_normalize(x_train)
    m = len(x_train)
    x_train = [[1] + x_train[i] for i in range(m)]
    iterations = 10000
    alpha = 0.01
    convergence_threshold = 0.000001
    theta, history_cost_iteration = gradient_descent(
        x_train, y_train, iterations, alpha, convergence_threshold
    )
    predictions = predict(x_test, x_feature_mean, x_feature_sdv, theta)
    print_accuracy(y_test, predictions)


if __name__ == "__main__":
    print("\nProcessing manual solution:\n")
    start = time()
    manual_solution()
    end = time()
    print(f"\nmanual_solution took {end - start}s to finish.")
    print("\nProcessing scikit learn solution:\n")
    start = time()
    scikit_learn_solution()
    end = time()
    print(f"\nscikit_learn_solution took {end - start}s to finish.\n")
```
