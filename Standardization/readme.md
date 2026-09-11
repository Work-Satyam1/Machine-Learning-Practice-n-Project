Machine Learning Notes — Scikit-learn & Data Preprocessing

This README contains my learning notes for the concepts I have practiced while working with Pandas, NumPy, Scikit-learn, train-test splitting, feature scaling, Linear Regression, predictions, and outliers.

1. Dataset and DataFrame

A Pandas DataFrame is a table-like data structure containing rows and columns.

import pandas as pd

df = pd.read_csv("data.csv")

Check the first few rows:

df.head()

Check the shape:

df.shape

Check column information:

df.info()

Check data types:

df.dtypes
2. Selecting Columns with iloc
df.iloc[:, 2:]
Meaning
       rows     columns
        ↓        ↓
df.iloc[:, 2:]
: → select all rows
2: → select columns from index 2 until the end

For example:

Index       0       1       2          3
          ID      Name     Age       Salary
df.iloc[:, 2:]

gives:

Age       Salary
Important

If you want to actually modify df:

df = df.iloc[:, 2:]

If you only write:

df.iloc[:, 2:]

you are only displaying/selecting the result. df itself does not change.

3. Features and Target

Suppose the dataset contains:

Age	EstimatedSalary	Purchased
19	19000	0
35	20000	0
48	90000	1
50	120000	1

Here:

Features (X)
X = df.drop('Purchased', axis=1)

This removes the Purchased column.

So:

X
↓
Age
EstimatedSalary
Target (y)
y = df['Purchased']

So:

y
↓
Purchased

The model learns:

Age + EstimatedSalary
          ↓
        Model
          ↓
      Purchased
4. train_test_split

Import:

from sklearn.model_selection import train_test_split

Split the dataset:

X_train, X_test, Y_train, Y_test = train_test_split(
    df.drop('Purchased', axis=1),
    df['Purchased'],
    test_size=0.3,
    random_state=0
)
What does it do?

It divides the dataset into:

                 Dataset
                    |
          ---------------------
          |                   |
       70%                     30%
     Training                 Testing
          |                     |
     X_train, Y_train      X_test, Y_test

Because:

test_size=0.3

means 30% test data and 70% training data.

train_test_split() parameters
test_size
test_size=0.3

30% of data goes to testing.

You can also give an exact number:

test_size=100

→ 100 rows for testing.

train_size
train_size=0.8

80% of the data goes to training.

Usually you don't need to specify both train_size and test_size.

random_state
random_state=0

Controls the random splitting.

Using the same value gives the same split every time.

random_state=0

and

random_state=42

are both valid.

The number itself has no special meaning.

shuffle

Default:

shuffle=True

It shuffles the data before splitting.

You can disable it:

shuffle=False
stratify

Useful for classification.

train_test_split(
    X,
    y,
    test_size=0.3,
    random_state=0,
    stratify=y
)

It tries to maintain the same class proportions in training and testing data.

For example:

Original:

Purchased = 0 → 60%
Purchased = 1 → 40%

Training:

0 → ~60%
1 → ~40%

Testing:

0 → ~60%
1 → ~40%
5. Checking Shape
X_train.shape
X_test.shape

or:

X_train.shape, X_test.shape

Example:

((280, 2), (120, 2))

means:

X_train
280 rows
2 features

X_test
120 rows
2 features
6. StandardScaler

Import:

from sklearn.preprocessing import StandardScaler

Create the scaler:

scaler = StandardScaler()

At this point, the scaler has not learned anything.

7. fit()
scaler.fit(X_train)

fit() means:

Learn the required parameters from the training data.

For StandardScaler, it learns:

Mean
Standard deviation

For example:

Age:
Mean = 37.86

EstimatedSalary:
Mean = 69807.14

You can check:

scaler.mean_

Example:

array([3.78642857e+01, 6.98071429e+04])

Scientific notation means:

3.78642857e+01 = 37.8642857

6.98071429e+04 = 69807.1429

So:

Age             → 37.8643
EstimatedSalary → 69807.14

These means are calculated from X_train, not the entire dataset.

8. StandardScaler Formula

StandardScaler uses:

$$ z = \frac{x-\mu}{\sigma} $$

Where:

x  = original value
μ  = mean
σ  = standard deviation
z  = scaled value

Example:

Suppose:

Age = 20
Mean Age = 40
Standard deviation = 10

Then:

$$ z = \frac{20-40}{10} $$ $$ z=-2 $$

So:

20 → -2
9. transform()

After learning the parameters:

scaler.fit(X_train)

we transform the data:

X_train_scaled = scaler.transform(X_train)

This applies the learned mean and standard deviation.

Then:

X_test_scaled = scaler.transform(X_test)

applies the same training parameters to the test data.

10. Why Don't We Use fit() on Test Data?

❌ Don't do:

scaler.fit(X_test)

Instead:

scaler.fit(X_train)

X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)

Why?

Because the test set should represent unseen data.

If we calculate scaling parameters from the test set, we allow information from the test set to influence preprocessing.

This is a form of data leakage.

Correct workflow:

X_train
   ↓
fit()
   ↓
Learn mean + standard deviation
   ↓
transform()
   ↓
X_train_scaled


X_test
   ↓
transform()
   ↓
Use SAME parameters learned from X_train
   ↓
X_test_scaled
11. NumPy Array vs Pandas DataFrame

After:

X_train_scaled = scaler.transform(X_train)

the result is generally a NumPy array.

It may look like:

[[-1.2, -0.8],
 [ 0.3,  0.5],
 [ 1.1,  1.4]]

Column names are not preserved in the same way as a DataFrame.

To convert it back:

X_train_scaled = pd.DataFrame(
    X_train_scaled,
    columns=X_train.columns
)

Similarly:

X_test_scaled = pd.DataFrame(
    X_test_scaled,
    columns=X_test.columns
)

Now you have:

Age	EstimatedSalary
-1.2	-0.8
0.3	0.5
1.1	1.4

instead of an unnamed NumPy array.

12. describe()
X_train.describe()

provides statistical information:

count
mean
std
min
25%
50%
75%
max

For example:

             Age   EstimatedSalary
count       280       280
mean         37.9     69807.1
std          10.2     34000.5
min          18       15000
25%          30       43000
50%          37       68000
75%          46       87000
max          60      150000
13. np.round()
np.round(X_train.describe(), 1)

np.round() rounds the values.

Example:

np.round(37.8642857, 1)

gives:

37.9

So:

np.round(X_train.describe(), 1)

means:

Show the statistical summary and round the values to 1 decimal place.

14. Checking Scaling

You can compare:

np.round(X_train.describe(), 1)

with:

np.round(X_train_scaled.describe(), 1)

Before scaling, you might see:

mean → 37.9

After scaling:

mean → 0.0

And the standard deviation should be approximately:

std → 1.0

This helps verify that StandardScaler worked correctly.

15. Visualizing Before and After Scaling
fig, (ax1, ax2) = plt.subplots(
    ncols=2,
    figsize=(12, 5)
)

ax1.scatter(
    X_train['Age'],
    X_train['EstimatedSalary']
)

ax1.set_title("Before Scaling")

ax2.scatter(
    X_train_scaled['Age'],
    X_train_scaled['EstimatedSalary'],
    color='red'
)

ax2.set_title("After Scaling")

plt.show()
What does this do?

It creates two plots:

┌─────────────────────┬─────────────────────┐
│                     │                     │
│   Before Scaling    │    After Scaling    │
│                     │                     │
└─────────────────────┴─────────────────────┘
Before scaling
ax1.scatter(
    X_train['Age'],
    X_train['EstimatedSalary']
)

X-axis:

Age

Y-axis:

EstimatedSalary
After scaling
ax2.scatter(
    X_train_scaled['Age'],
    X_train_scaled['EstimatedSalary']
)

Now both features have been standardized.

The values change, but the overall relationship/pattern remains similar.

16. Machine Learning Model

Example:

from sklearn.linear_model import LinearRegression

lr = LinearRegression()

Train it:

lr.fit(X_train, Y_train)

fit() means:

Learn the relationship between X_train and Y_train.

For Linear Regression, the model learns an equation like:

$$ y=b_0+b_1x_1+b_2x_2 $$

For your dataset:

$$ Purchased = b_0+ b_1(Age)+ b_2(EstimatedSalary) $$
17. Model Coefficients

You can inspect what the model learned:

lr.coef_

This gives the coefficients.

And:

lr.intercept_

gives the intercept.

Conceptually, suppose:

Age coefficient = 0.02
Salary coefficient = 0.00001
Intercept = -1.5

The model has learned:

$$ Purchased = -1.5+ 0.02(Age)+ 0.00001(Salary) $$
18. predict()

After training:

lr.fit(X_train, Y_train)

you can make predictions:

y_pred = lr.predict(X_test)

This means:

Take the unseen X_test data and use the relationship learned by lr to calculate predictions.

The flow is:

X_train + Y_train
       ↓
    fit()
       ↓
Model learns relationship
       ↓
    Trained Model
       ↓
    predict()
       ↑
     X_test
       ↓
    y_pred
19. How Prediction Actually Happens

Suppose the model learned:

$$ y=-1.5+0.02(Age)+0.00001(Salary) $$

A new person has:

Age = 40
Salary = 80000

The model calculates:

$$ y=-1.5+(0.02)(40)+(0.00001)(80000) $$ $$ y=-1.5+0.8+0.8 $$ $$ y=0.1 $$

So the model predicts:

0.1

This is essentially what predict() is doing internally: using the learned parameters to calculate an output for new input data.

20. Scaled Model

You can also train a model using scaled data:

lr_scaled = LinearRegression()

lr_scaled.fit(
    X_train_scaled,
    Y_train
)

Now the model learns a relationship using:

Scaled Age
Scaled EstimatedSalary

instead of:

Original Age
Original EstimatedSalary

Then:

y_pred_scaled = lr_scaled.predict(X_test_scaled)

The flow becomes:

X_train
   ↓
StandardScaler
   ↓
X_train_scaled
   ↓
lr_scaled.fit()
   ↓
Trained scaled model
   ↓
X_test_scaled
   ↓
predict()
   ↓
y_pred_scaled
21. Important Rule

If the model was trained with scaled data:

lr_scaled.fit(X_train_scaled, Y_train)

then give it scaled data when predicting:

lr_scaled.predict(X_test_scaled)

Not:

lr_scaled.predict(X_test)  # ❌

The model expects the same type of input representation it saw during training.

22. fit(), transform(), predict()

Remember these three words:

Method	Meaning
fit()	Learn
transform()	Apply transformation
predict()	Make prediction

Example:

scaler.fit(X_train)

Scaler learns.

X_train_scaled = scaler.transform(X_train)

Scaler transforms.

lr.fit(X_train_scaled, Y_train)

Model learns.

y_pred = lr.predict(X_test_scaled)

Model predicts.

23. Adding Rows to a DataFrame

You tried:

df = df.append(
    pd.DataFrame({
        'Age': [5, 90, 95],
        'EstimatedSalary': [1000, 250000, 350000],
        'Purchased': [0, 1, 1]
    }),
    ignore_index=True
)

This creates three new rows:

Age	EstimatedSalary	Purchased
5	1,000	0
90	250,000	1
95	350,000	1

These are unusual values and can be used to demonstrate outliers.

24. Modern Pandas: append() vs concat()

DataFrame.append() has been removed from modern Pandas versions.

Use:

new_data = pd.DataFrame({
    'Age': [5, 90, 95],
    'EstimatedSalary': [1000, 250000, 350000],
    'Purchased': [0, 1, 1]
})

df = pd.concat(
    [df, new_data],
    ignore_index=True
)
ignore_index=True

This tells Pandas to create a fresh sequential index:

0
1
2
3
4
...

instead of preserving the indexes from the two DataFrames.

25. Categorical Data Problem

If your dataset contains:

Gender
------
Male
Female
Male

and you run:

scaler.fit(X_train)

you can get:

ValueError: could not convert string to float: 'Male'

Why?

StandardScaler requires numerical values.

It cannot calculate:

$$ \frac{Male-Mean}{StandardDeviation} $$

because "Male" is a string.

You need to encode categorical values into numbers before applying numerical scaling.

For example:

df['Gender'] = df['Gender'].map({
    'Male': 0,
    'Female': 1
})

Then:

Male   → 0
Female → 1
26. Complete Basic Workflow

A basic preprocessing and ML workflow looks like this:

import pandas as pd
import numpy as np

from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LinearRegression
Load data
df = pd.read_csv("data.csv")
Select required columns
df = df.iloc[:, 2:]
Separate X and y
X = df.drop('Purchased', axis=1)
y = df['Purchased']
Split
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.3,
    random_state=0
)
Scale
scaler = StandardScaler()

scaler.fit(X_train)

X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)
Convert back to DataFrame if needed
X_train_scaled = pd.DataFrame(
    X_train_scaled,
    columns=X_train.columns
)

X_test_scaled = pd.DataFrame(
    X_test_scaled,
    columns=X_test.columns
)
Train model
lr_scaled = LinearRegression()

lr_scaled.fit(
    X_train_scaled,
    y_train
)
Predict
y_pred_scaled = lr_scaled.predict(X_test_scaled)
27. The Big Picture

The entire process can be remembered as:

                  RAW DATA
                     │
                     ↓
              Clean / Prepare
                     │
                     ↓
              Separate X and y
                     │
                     ↓
             Train / Test Split
                     │
             ┌───────┴───────┐
             ↓               ↓
          Training          Testing
             │               │
             ↓               ↓
          fit scaler      transform
             │               │
             ↓               ↓
       X_train_scaled    X_test_scaled
             │               │
             └───────┬───────┘
                     ↓
                 Train Model
                     │
                     ↓
                  predict()
                     │
                     ↓
                  y_pred
                     │
                     ↓
              Evaluate Model