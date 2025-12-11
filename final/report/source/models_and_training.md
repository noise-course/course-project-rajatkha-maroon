# 5. Models and training
With the PCA-reduced datasets, I now train several classical ML models without running into memory issues.

## 5.1. Train and test splits
I perform a **single stratified train/validation split** for each task:

```python
from sklearn.model_selection import train_test_split

# EASY class
X_train_easy, X_val_easy, y_train_easy, y_val_easy = train_test_split(
    X_easy, y_easy, test_size=0.2, stratify=y_easy, random_state=42
)

# HARD class
X_train_hard, X_val_hard, y_train_hard, y_val_hard = train_test_split(
    X_hard, y_hard, test_size=0.2, stratify=y_hard, random_state=42
)
```

## 5.2. Models
I evaluate the following models:

1. Logistic Regression (with StandardScaler)
2. SGDClassifier (linear, log-loss)
3. RandomForestClassifier
4. GradientBoostingClassifier
5. XGBClassifier

```python
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression, SGDClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
import numpy as np

def get_models():
    models = {}

    models["RandomForest"] = RandomForestClassifier(
        n_estimators=300,
        max_depth=None,
        n_jobs=-1,
        random_state=42
    )

    models["LogReg"] = make_pipeline(
        StandardScaler(),
        LogisticRegression(max_iter=2000)
    )

    models["SGD_logreg"] = make_pipeline(
        StandardScaler(),
        SGDClassifier(loss="log_loss", max_iter=1000, tol=1e-3)
    )

    models["GradBoost"] = GradientBoostingClassifier(
        n_estimators=200,
        random_state=42
    )

    if HAS_XGB:
        models["XGBoost"] = XGBClassifier(
            n_estimators=300,
            max_depth=6,
            learning_rate=0.1,
            subsample=0.8,
            colsample_bytree=0.8,
            tree_method="hist",
            eval_metric="logloss",
            n_jobs=-1
        )
    return models
```
