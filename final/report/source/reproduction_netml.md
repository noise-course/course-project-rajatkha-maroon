# 6. netML Autogluon

I also ran netML's Autogluon model to try to reproduce the results (maybe better). I ran the following training over both easy and hard datasets. I show for easy below.

```python
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, balanced_accuracy_score

train_data_ag = X_train_easy.copy()
train_data_ag["label"] = y_train_easy

from autogluon.tabular import TabularPredictor

predictor = TabularPredictor(
    label="label",
    problem_type="multiclass",
    verbosity=0,
)

_ = predictor.fit(
    train_data=train_data_ag,
    time_limit=300,
    presets="medium_quality_faster_train",
    verbosity=0,
)

y_pred = predictor.predict(X_val_easy)

print("Autogluon accuracy:         ",
      accuracy_score(y_val_easy, y_pred))
print("Autogluon balanced accuracy:",
      balanced_accuracy_score(y_val_easy, y_pred))
```

## 6.1 Evaluation

1. For easy: Autogluon accuracy: 0.9999699067108034, Autogluon balanced accuracy: 0.9999419144981412
2. For hard: Autogluon accuracy: 0.9123683418597652, Autogluon balanced accuracy: 0.7536582945717588

## 6.2 Interpretation
The benchmarks reports balanced accuracy 0.80+ for the best model, whereas my Autogluon score is 0.7536 balanced accuracy. I think this might be attributed to the following reasons:

1. While PCA preserves global variance, it does not preserve: fine-grained patterns, and fine differences in small classes.
2. Flow-generation mismatch with official pipeline: It might be possible that flow generation is mismatched with the original pipeline.
