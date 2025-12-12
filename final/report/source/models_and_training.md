# 5. Models and training
With the PCA-reduced datasets, I now train several classical ML models without running into memory issues.

## 5.1. Train and test splits
I perform a **single stratified train/validation split** for each task:

```python
from sklearn.model_selection import train_test_split

X_train_easy, X_val_easy, y_train_easy, y_val_easy = train_test_split(
    X_easy, y_easy, test_size=0.2, stratify=y_easy, random_state=42
)

X_train_hard, X_val_hard, y_train_hard, y_val_hard = train_test_split(
    X_hard, y_hard, test_size=0.2, stratify=y_hard, random_state=42
)
```

## 5.2. Models
I evaluate the following models:

1. Random forest
2. LogReg
3. SGD LogReg

## 5.3. Evaluation

**Easy results:**

   task         model      acc  balanced_acc  precision_macro  recall_macro  f1_macro  

0  easy  RandomForest  0.99997      0.999942         0.999980      0.999942  0.999961   

1  easy        LogReg  0.99994      0.999909         0.999934      0.999909  0.999922  

2  easy    SGD_logreg  0.99994      0.999909         0.999934      0.999909  0.999922   


**Hard results:**

   task         model       acc  balanced_acc  precision_macro  recall_macro  f1_macro
   
0  hard  RandomForest  0.884492      0.703461         0.707550      0.703461  0.705189  

1  hard        LogReg  0.832160      0.583881         0.669906      0.583881  0.594641 

2  hard    SGD_logreg  0.796339      0.522495         0.587273      0.522495  0.525238  

## 5.3. Interpretation for easy class

All three models achieve virtually perfect performance (balanced accuracy ≥ 0.9999).
This indicates:

1. The PCA representation preserves almost all the discriminative information needed to distinguish benign from malware flows.
2. The binary decision boundary is extremely well-structured, meaning that even linear models (LogReg, SGD) find a clean separation between the two classes.
3. The dataset at the easy-label level is highly linearly separable, at least after nPrint hashing + PCA projection.

## 5.4. Interpretation for hard class
Results are significantly lower than the easy task.

Key observations:

1. Random Forest performs best on the hard task: achieves 88% accuracy, but only 70% balanced accuracy. This gap means the model favors the majority families and struggles on minority ones.
2. Linear models perform noticeably worse. Logistic Regression: 83% accuracy but only 58% balanced accuracy. SGD: drops further to ~52% balanced accuracy. This indicates that malware families are not linearly separable in 15-dimensional PCA space.
3. Macro precision/recall reveal imbalance issues: Macro averaging gives each class equal weight. The drop from accuracy to macro F1 suggests that some families dominate the dataset while others are rare.

## 5.5. Testing other models (v2)
On the hard class, I also evaluate the following models:

1. ExtraTrees
2. HistGB
3. SGD_LogReg
4. SVM_rbf

## 5.6. Evaluation on other models (v2)
After introducing additional models better suited to nonlinear and high-dimensional hashed feature spaces, the hard-task results improved noticeably. The new metrics are:

Model	             Accuracy	Balanced Acc	Precision (macro)	Recall (macro)	F1 (macro)

ExtraTrees	         0.8813	    0.6937	        0.7192	            0.6937	        0.6974

HistGradientBoosting 0.9010	    0.7329	        0.7081	            0.7329	        0.7070

SVM_rbf              0.8825     0.7741          0.7685              0.7741          0.7147

## 5.7. Intepretation on other models (v2)
The standout model is the SVM with RBF kernel, which achieves 0.7741 balanced accuracy, the highest among all tested methods. Unlike tree models, the RBF kernel constructs flexible, curved decision boundaries in the compressed 15-dimensional space and is better able to isolate smaller malware families from the dominant clusters. This leads to significantly higher recall on minority classes, which directly improves balanced accuracy. The SVM result is also the closest to the official NetML leaderboard performance, indicating that kernel-based methods can partially compensate for the information loss introduced by PCA.
