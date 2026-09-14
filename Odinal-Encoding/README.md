# Day 26 — Ordinal Encoding & Label Encoding

This notebook explains how categorical data can be converted into numerical values using **Ordinal Encoding** and **Label Encoding**.

Machine learning models generally work with numerical data, so categorical values such as `Poor`, `Average`, `Good`, `School`, `UG`, and `PG` need to be converted into numbers.

---

## Topics Covered

* Categorical Data
* Ordinal Encoding
* Label Encoding
* `fit()`
* `transform()`
* `fit_transform()`
* `categories_`
* `classes_`
* Encoding training and testing data
* Difference between feature encoding and target encoding

---

## Dataset

The dataset contains the following columns:

| Column      | Type        | Description                                |
| ----------- | ----------- | ------------------------------------------ |
| `age`       | Numerical   | Age of the customer                        |
| `gender`    | Categorical | Gender of the customer                     |
| `review`    | Ordinal     | Customer review                            |
| `education` | Ordinal     | Education level                            |
| `purchased` | Target      | Whether the customer purchased the product |

Example:

| age | gender | review  | education | purchased |
| --: | ------ | ------- | --------- | --------- |
|  28 | Male   | Good    | UG        | Yes       |
|  35 | Female | Poor    | PG        | No        |
|  22 | Male   | Average | School    | Yes       |

---

# 1. Ordinal Encoding

Ordinal encoding is used when categorical values have a **meaningful order or ranking**.

For example:

```text
Poor < Average < Good
```

We can represent them as:

```text
Poor     → 0
Average  → 1
Good     → 2
```

Similarly, education has an order:

```text
School < UG < PG
```

So:

```text
School → 0
UG     → 1
PG     → 2
```

The important point is that the numerical values represent the original order.

---

## Using OrdinalEncoder

```python
from sklearn.preprocessing import OrdinalEncoder

oe = OrdinalEncoder(
    categories=[
        ['Poor', 'Average', 'Good'],
        ['School', 'UG', 'PG']
    ]
)
```

Here, the first list corresponds to the `review` column and the second list corresponds to the `education` column.

The mappings are therefore:

```text
review

Poor     → 0
Average  → 1
Good     → 2
```

```text
education

School → 0
UG     → 1
PG     → 2
```

---

# 2. `fit()`

```python
oe.fit(X_train)
```

`fit()` learns the categories and prepares the encoder.

It does **not** convert the data.

Think of it as:

```text
fit()
  ↓
Learn the mapping
```

For example:

```text
Poor → 0
Average → 1
Good → 2
```

---

# 3. `transform()`

```python
X_train = oe.transform(X_train)
```

`transform()` applies the mapping learned during `fit()`.

For example:

```text
Before:

Good     UG
Poor     PG
Average  School
```

After:

```text
2  1
0  2
1  0
```

---

# 4. `fit_transform()`

Instead of writing:

```python
oe.fit(X_train)
X_train = oe.transform(X_train)
```

we can use:

```python
X_train = oe.fit_transform(X_train)
```

`fit_transform()` performs both operations:

```text
fit()
  ↓
learn mapping
  ↓
transform()
  ↓
convert data
```

---

# 5. `categories_`

After fitting the encoder, we can inspect the learned categories using:

```python
oe.categories_
```

Example:

```python
[
    array(['Poor', 'Average', 'Good'], dtype=object),
    array(['School', 'UG', 'PG'], dtype=object)
]
```

The first array belongs to `review`.

The second array belongs to `education`.

So:

```text
review:
Poor     → 0
Average  → 1
Good     → 2
```

and:

```text
education:
School → 0
UG     → 1
PG     → 2
```

The underscore in `categories_` indicates that this information is available after fitting the encoder.

---

# 6. Why Explicit Category Order Is Important

For ordinal data, the order matters.

Instead of allowing the encoder to determine the order automatically, we explicitly provide it:

```python
categories=[
    ['Poor', 'Average', 'Good'],
    ['School', 'UG', 'PG']
]
```

This ensures that:

```text
Poor < Average < Good
```

and:

```text
School < UG < PG
```

are represented correctly.

---

# 7. Label Encoding

`LabelEncoder` is generally used to encode the **target variable (`y`)**.

In this dataset:

```text
purchased
```

is the target.

It contains:

```text
Yes
No
```

We can encode it as:

```text
No  → 0
Yes → 1
```

---

## Using LabelEncoder

```python
from sklearn.preprocessing import LabelEncoder

le = LabelEncoder()
```

First, fit it:

```python
le.fit(y_train)
```

Then inspect the learned classes:

```python
le.classes_
```

Output:

```python
array(['No', 'Yes'], dtype=object)
```

Therefore:

```text
No  → 0
Yes → 1
```

---

# 8. Transforming the Target

```python
y_train = le.transform(y_train)
y_test = le.transform(y_test)
```

Example:

Before:

```text
['Yes', 'No', 'No', 'Yes']
```

After:

```text
[1, 0, 0, 1]
```

The same encoder is used for both training and testing data.

---

# 9. Complete Encoding Flow

```text
                    DATASET
                       │
                       ▼
        ┌───────────────────────────┐
        │ review                    │
        │ education                 │
        │ purchased                 │
        └───────────────────────────┘
                       │
                 Train/Test Split
                       │
              ┌────────┴────────┐
              ▼                 ▼
              X                 y
       Input Features         Target
              │                 │
              ▼                 ▼
      OrdinalEncoder       LabelEncoder
              │                 │
              ▼                 ▼
       review:             No  → 0
       Poor → 0            Yes → 1
       Average → 1
       Good → 2
       
       education:
       School → 0
       UG → 1
       PG → 2
```

---

# 10. Complete Example

```python
from sklearn.preprocessing import OrdinalEncoder
from sklearn.preprocessing import LabelEncoder

# Ordinal Encoding
oe = OrdinalEncoder(
    categories=[
        ['Poor', 'Average', 'Good'],
        ['School', 'UG', 'PG']
    ]
)

X_train = oe.fit_transform(X_train)
X_test = oe.transform(X_test)

# Check learned categories
print(oe.categories_)

# Label Encoding
le = LabelEncoder()

y_train = le.fit_transform(y_train)
y_test = le.transform(y_test)

# Check learned classes
print(le.classes_)
```

---

# 11. `fit()` vs `transform()` vs `fit_transform()`

| Function          | Purpose                        |
| ----------------- | ------------------------------ |
| `fit()`           | Learns the mapping             |
| `transform()`     | Applies the learned mapping    |
| `fit_transform()` | Learns and applies the mapping |

Example:

```python
oe.fit(X_train)
```

means:

```text
Learn
```

```python
oe.transform(X_train)
```

means:

```text
Convert
```

```python
oe.fit_transform(X_train)
```

means:

```text
Learn + Convert
```

---

# 12. Important Rule for Train and Test Data

The encoder should be fitted on the training data.

Correct:

```python
X_train = oe.fit_transform(X_train)
X_test = oe.transform(X_test)
```

Incorrect:

```python
X_train = oe.fit_transform(X_train)
X_test = oe.fit_transform(X_test)
```

Why?

Because the test data should use the **same mapping learned from the training data**.

The same idea applies to the target:

```python
y_train = le.fit_transform(y_train)
y_test = le.transform(y_test)
```

---

# 13. Ordinal Encoding vs Label Encoding

| Feature          | OrdinalEncoder              | LabelEncoder                      |
| ---------------- | --------------------------- | --------------------------------- |
| Mainly used for  | Input features              | Target labels                     |
| Input            | One or more feature columns | Target column                     |
| Example          | `Poor`, `Average`, `Good`   | `No`, `Yes`                       |
| Multiple columns | Yes                         | No, intended for one target array |
| Output           | Numerical values            | Numerical class labels            |

---

# 14. When Should You Use Ordinal Encoding?

Use ordinal encoding when the categories have a meaningful order.

Examples:

```text
Poor < Average < Good
```

```text
Low < Medium < High
```

```text
School < UG < PG
```

Do **not** use ordinal encoding simply because a column contains strings.

For example:

```text
Male
Female
```

has no natural order.

Similarly:

```text
Red
Blue
Green
```

does not have a meaningful numerical ranking.

For such nominal categories, **One-Hot Encoding** is generally more appropriate.

---

# Key Takeaways

```text
Categorical Data
       ↓
Need numerical representation
       ↓
Choose encoding based on the type of category
```

### OrdinalEncoder

Used for ordered categories:

```text
Poor → 0
Average → 1
Good → 2
```

### LabelEncoder

Usually used for target labels:

```text
No → 0
Yes → 1
```

### Functions

```text
fit()           → Learn
transform()     → Convert
fit_transform() → Learn + Convert
```

### Inspect learned values

```python
oe.categories_
```

for `OrdinalEncoder`.

```python
le.classes_
```

for `LabelEncoder`.

The most important concept is not the syntax. It is understanding **whether the categorical values actually have an order** before choosing ordinal encoding.
