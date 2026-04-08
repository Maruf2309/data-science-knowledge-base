
# Handing Missing Values
```bash

import pandas as pd
import numpy as np

# Sample dirty data
df = pd.DataFrame({
    'Age': [25, np.nan, 30, 28, 35, np.nan],
    'Salary': [50000, 60000, np.nan, 55000, 70000, 52000],
    'City': ['NY', 'LA', 'NY', np.nan, 'LA', 'NY']
})

# 1. Row Elimination
df_clean_rows = df.dropna()

# 2. Column Elimination (if >50% missing? none here)
# 3. Mean Imputation
df['Age'].fillna(df['Age'].mean(), inplace=True)

# 4. Mode Imputation
df['City'].fillna(df['City'].mode()[0], inplace=True)

print(df)
```
### 1. SimpleImputer (Mean/Median/Mode):
```bash

from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy='median')
imputer.fit(X_train)  # FIT on TRAIN only
X_train = imputer.transform(X_train)
X_test = imputer.transform(X_test)  # TRANSFORM test with TRAIN statistics
```
### 2. KNNImputer:

```bash
from sklearn.impute import KNNImputer

imputer = KNNImputer(n_neighbors=5)
imputer.fit(X_train)  # KNN distances calculated from TRAIN only
X_train = imputer.transform(X_train)
X_test = imputer.transform(X_test)  # Test neighbors found in TRAIN space
```

### 3. IterativeImputer:
```bash
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer

imputer = IterativeImputer(max_iter=10)
imputer.fit(X_train)  # Regression models learned from TRAIN only
X_train = imputer.transform(X_train)
X_test = imputer.transform(X_test)  # Test values predicted using TRAIN models
```
## Pipeline for Automation
```bash
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.ensemble import RandomForestClassifier

# Pipeline automatically handles fit on train, transform on test
pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('classifier', RandomForestClassifier())
])

# Fit on TRAIN only
pipeline.fit(X_train, y_train)

# Transform TEST automatically (using TRAIN statistics)
predictions = pipeline.predict(X_test)
```
### Pipeline-এর সুবিধা:
```bash
✅ Automatically কোন Data Leakage হবে না
✅ Cross-validation-এ সঠিকভাবে কাজ করে
✅ Code clean ও maintainable হয়

```
### Checklist
```bash
□ Imputer তৈরি করলাম
□ Imputer .fit() করলাম → শুধু X_train-এর উপর
□ X_train transform করলাম → .transform(X_train)
□ X_test transform করলাম → .transform(X_test) (একই imputer দিয়ে)
□ কখনোই imputer.fit(X_test) করব না
□ Production-এ গিয়ে নতুন ডাটা এলে → শুধু .transform() করব, .fit() নয়
```
### Memory Ticks:
```bash
Train = Student (পড়ে)
Test = Exam (পরীক্ষা)

ভুল: Exam-এর question দেখে Student পড়বে → Cheating
সঠিক: Student শুধু বই (Train) পড়ে Exam দেয় → Realistic

Imputer = বই
Test data দেখে Imputer লিখলে = Cheating
```
### Conclusion
```bash
Golden Rule of Imputation:

FIT on TRAIN only
TRANSFORM both TRAIN and TEST

কখনোই FIT on full data করবেন না।
কখনোই FIT on TEST করবেন না।

Pipeline use করুন → Automatically correct হবে।

```
## Data Leakage on Imputation
```bash
❌ 90% beginners make this mistake.

They impute missing values BEFORE splitting train-test.

And then wonder why their model fails in production.

👇 Let me explain why this is DEAD WRONG.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🚫 The WRONG way:

Full Data → Calculate Mean → Impute → Split

❌ Problem: Test data influences Train data
❌ Result: Data Leakage + Overfitting
❌ Reality: Model looks good on paper, fails in real world

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ The RIGHT way:

Full Data → Split → Fit Imputer on TRAIN only → Transform BOTH

✓ Train learns from Train only
✓ Test is completely unseen
✓ Realistic performance evaluation

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💻 Quick Example:

from sklearn.impute import SimpleImputer

# ✅ CORRECT
imputer = SimpleImputer(strategy='median')
imputer.fit(X_train)              # Learn from TRAIN only
X_train = imputer.transform(X_train)
X_test = imputer.transform(X_test)  # Use TRAIN statistics

# ❌ WRONG
imputer.fit(X)  # NO! Full data leakage!

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎯 The Golden Rule to remember:

FIT on TRAIN only
TRANSFORM both TRAIN and TEST

Or better yet — use Pipeline:

Pipeline([
    ('imputer', SimpleImputer()),
    ('model', RandomForest())
]).fit(X_train, y_train)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

💡 Think of it this way:

Train = Student studying textbooks
Test = Final exam

If you show exam questions BEFORE studying → That's CHEATING.

Same with imputation. Test data should NEVER touch your imputer's statistics.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⚠️ This applies to ALL imputation methods:

• Mean/Median/Mode Imputation
• KNN Imputer
• Iterative Imputer
• Even Simple Deletion!

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Quick Checklist:

□ Did I split before imputation?
□ Did I .fit() only on X_train?
□ Did I use the SAME imputer for X_test?
□ Did I avoid .fit() on X_test?

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎯 Your turn:

Have you ever made this mistake? (Be honest 😄)

What's your go-to method to prevent data leakage?

👇 Comment below!

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

♻️ Repost to help others avoid this common mistake.

🔔 Follow me for more Data Science tips.

#DataScience #MachineLearning #Python #DataLeakage #FeatureEngineering #MissingData #DataScienceTips #LearnDataScience