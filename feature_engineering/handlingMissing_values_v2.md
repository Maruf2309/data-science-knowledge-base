
# Handling Missing Values (Wrong & Correct Method)
## ❌ Wrong Method
``` bash
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.impute import SimpleImputer

# Sample Data
df = pd.DataFrame({
    'Age': [25, 30, np.nan, 35, 40, np.nan, 28, 32, 38, 45],
    'Salary': [50000, 60000, 55000, 70000, 65000, 72000, 58000, 62000, 68000, 75000]
})

# ❌ Wrong: Split-এর আগে Imputation
imputer = SimpleImputer(strategy='mean')
df_imputed = imputer.fit_transform(df)  # Getting mean from full data(Including Mean) / Wrong  

# Then Split
X_train, X_test = train_test_split(df_imputed, test_size=0.2, random_state=42)

# সমস্যা: X_test-এর Age values X_train-এর Mean-এ influence ফেলেছে!
```
## ✅ Correct Method
``` bash
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
from sklearn.impute import SimpleImputer

# ডামি ডাটা
df = pd.DataFrame({
    'Age': [25, 30, np.nan, 35, 40, np.nan, 28, 32, 38, 45],
    'Salary': [50000, 60000, 55000, 70000, 65000, 72000, 58000, 62000, 68000, 75000]
})

# ✅ Correct : First Split
X = df[['Age', 'Salary']]  # Features
X_train, X_test = train_test_split(X, test_size=0.2, random_state=42)

# Train Set-এ Imputer FIT করি
imputer = SimpleImputer(strategy='mean')
imputer.fit(X_train)  # শুধু TRAIN data থেকে Mean বের করব

# Train ও Test উভয়কেই TRANSFORM করি
X_train_imputed = imputer.transform(X_train)
X_test_imputed = imputer.transform(X_test)

# এখন X_test-এর কোনো influence X_train-এ নেই!
print(f"Train Imputer Mean: {imputer.statistics_[0]}")  # শুধু Train data থেকে calculated
```

### ❌ ভুল পদ্ধতি (Data Leakage):
``` bash
পুরো ডাটাসেট (1000 rows)
        │
        ▼
Mean Imputation (Age কলামের Mean বের করলাম = 35)
        │
        ▼
Train-Test Split (80% Train, 20% Test)
        │
        ├──► Train Set (800 rows) → Mean = 35 (Test-এর influence আছে!)
        │
        └──► Test Set (200 rows) → Mean = 35

সমস্যা: Test Set-এর Age values Train Set-এর Mean calculation-এ influence ফেলেছে!
```
### ✅ সঠিক পদ্ধতি (No Leakage):
``` bash
পুরো ডাটাসেট (1000 rows)
        │
        ▼
Train-Test Split (80% Train, 20% Test)
        │
        ├──► Train Set (800 rows) → Calculate Mean = 34
        │         │
        │         ▼
        │    Fit Imputer on TRAIN only
        │         │
        │         ▼
        │    Transform TRAIN
        │
        └──► Test Set (200 rows)
                  │
                  ▼
             Transform TEST using TRAIN's Mean (=34)

Result: Set-এর কোনো influence Train Set-এ নেই!
```
### Story to Remember
``` bash
আপনি যদি ট্রেন ও টেস্ট ডাটা আলাদা করার আগে ইম্পিউটেশন করেন, 
তবে টেস্ট ডাটার তথ্য ট্রেন ডাটাতে লিক হয়ে যাবে। 
এটি আপনার মডেলকে "চিটিং" করিয়ে দেবে।
```

## Real Example:
```bash

ধরুন, আপনার Test Set-এ Age-এর মান অনেক বেশি (যেমন: ৮০, ৯০, ১০০)।

❌ ভুল পদ্ধতি (Split-এর আগে Imputation):
-----------------------------------------------------------------
পুরো ডাটার Mean Age = 50 (Test-এর high values influence ফেলেছে)
Train Set-এর Mean Age = 50 (যেখানে আসলে Train-এর Mean ছিল 35)

→ মডেল শিখল: "গড় বয়স ৫০"
→ Test Set-এ গিয়ে ভুল করবে!

✅ সঠিক পদ্ধতি (Split-এর পরে Imputation):
-----------------------------------------------------------------
Train Set-এর Mean Age = 35 (Test-এর influence নেই)
Train Set Imputation = 35 দিয়ে fill

→ মডেল শিখল: "গড় বয়স ৩৫"
→ Test Set-এ গিয়ে Realistic performance দেখাবে