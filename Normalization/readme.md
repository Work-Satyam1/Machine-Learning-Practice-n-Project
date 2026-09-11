# Feature Normalization Using Min-Max Scaling

This project demonstrates **Feature Normalization** using the **Min-Max Scaling** technique on the Wine dataset.

The main purpose of the project is to understand why normalization is required in Machine Learning, how it works, and how it changes the scale of features without changing their relative distribution.

## Objective

The project focuses on:

* Understanding Feature Normalization
* Understanding why features need to be normalized
* Visualizing feature distributions before normalization
* Splitting data into training and testing sets
* Applying Min-Max Normalization
* Comparing features before and after normalization
* Understanding the importance of fitting the scaler only on training data

## Dataset

The project uses the **Wine dataset** stored in:

```text
wine_data.csv
```

For this project, three columns are selected:

| Column      | Description              |
| ----------- | ------------------------ |
| Class label | Target class of the wine |
| Alcohol     | Alcohol content          |
| Malic acid  | Malic acid content       |

The dataset contains:

```text
178 rows × 3 columns
```

### Input Features

```text
Alcohol
Malic acid
```

### Target

```text
Class label
```

## Technologies Used

* Python
* NumPy
* Pandas
* Matplotlib
* Seaborn
* Scikit-learn
* Jupyter Notebook / VS Code

## What is Normalization?

**Normalization** is a feature scaling technique used to bring numerical features into a common range.

Different features can have very different ranges.

For example:

```text
Alcohol      → 11.0 – 14.8
Malic acid   → 0.9 – 5.6
```

Because the ranges are different, some Machine Learning algorithms may give more importance to a feature simply because of its numerical scale.

Normalization helps bring these features to a comparable scale.

## Min-Max Normalization

This project uses **Min-Max Scaling** for normalization.

The formula is:

```text
X_normalized = (X - X_min) / (X_max - X_min)
```

The resulting values are generally between:

```text
0 and 1
```

### Example

Suppose:

```text
Minimum = 10
Maximum = 20
Value = 15
```

Then:

```text
X_normalized = (15 - 10) / (20 - 10)

             = 5 / 10

             = 0.5
```

Therefore:

```text
15 → 0.5
```

## Project Workflow

```text
Wine Dataset
      ↓
Data Loading
      ↓
Data Exploration
      ↓
Feature Visualization
      ↓
Train-Test Split
      ↓
Fit Min-Max Scaler
      ↓
Normalize Features
      ↓
Compare Before & After
      ↓
Analyze Results
```

## 1. Loading the Dataset

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv(
    'wine_data.csv',
    header=None,
    usecols=[0, 1, 2]
)

df.columns = ['Class label', 'Alcohol', 'Malic acid']
```

The resulting DataFrame contains 178 rows and 3 columns.

## 2. Visualizing Feature Distributions

KDE plots are used to understand the distribution of the features.

### Alcohol

```python
sns.kdeplot(df['Alcohol'])
```

### Malic Acid

```python
sns.kdeplot(df['Malic acid'])
```

These visualizations show how the values of each feature are distributed.

## 3. Visualizing Different Classes

A scatter plot is used to visualize the relationship between Alcohol and Malic acid for different wine classes.

```python
color_dict = {
    1: 'red',
    2: 'blue',
    3: 'green'
}

sns.scatterplot(
    data=df,
    x='Alcohol',
    y='Malic acid',
    hue='Class label',
    palette=color_dict
)
```

This helps visualize how the different classes are distributed based on the selected features.

## 4. Train-Test Split

Before normalization, the dataset is divided into training and testing data.

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    df.drop('Class label', axis=1),
    df['Class label'],
    test_size=0.3,
    random_state=0
)
```

The data is divided into:

```text
Training data → 124 rows
Testing data  → 54 rows
```

## 5. Applying Min-Max Normalization

The `MinMaxScaler` from Scikit-learn is used.

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()

scaler.fit(X_train)

X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

The scaled arrays are converted back into DataFrames:

```python
X_train_scaled = pd.DataFrame(
    X_train_scaled,
    columns=X_train.columns
)

X_test_scaled = pd.DataFrame(
    X_test_scaled,
    columns=X_test.columns
)
```

## 6. Before Normalization

The original training data has different ranges.

```text
Alcohol
Minimum → approximately 11
Maximum → approximately 14.8

Malic acid
Minimum → approximately 0.9
Maximum → approximately 5.6
```

Because the features have different ranges, their numerical scales are different.

## 7. After Normalization

After applying Min-Max Normalization:

```text
Alcohol
Minimum → 0
Maximum → 1

Malic acid
Minimum → 0
Maximum → 1
```

The features are now represented on the same scale.

## 8. Comparing Before and After Normalization

```python
fig, (ax1, ax2) = plt.subplots(
    ncols=2,
    figsize=(12, 5)
)

ax1.scatter(
    X_train['Alcohol'],
    X_train['Malic acid'],
    c=y_train
)

ax1.set_title("Before Normalization")

ax2.scatter(
    X_train_scaled['Alcohol'],
    X_train_scaled['Malic acid'],
    c=y_train
)

ax2.set_title("After Normalization")

plt.show()
```

### Observation

The numerical range changes, but the overall relationship between the features remains similar.

```text
Before:

Alcohol      → 11 – 14.8
Malic acid   → 0.9 – 5.6


After:

Alcohol      → 0 – 1
Malic acid   → 0 – 1
```

Normalization changes the **scale**, not the underlying ordering of the values.

## 9. Comparing Distributions

The distributions can also be compared using KDE plots.

### Alcohol

```python
fig, (ax1, ax2) = plt.subplots(
    ncols=2,
    figsize=(12, 5)
)

ax1.s
```
