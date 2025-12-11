# 8. Conclusion, discussion, and future work

## 8.1 Conclusion and discussion

This project explored an end-to-end malware traffic classification pipeline:

1. Starting from the NetML Malware pcapML file,
2. Extracting flows and labels,
3. Generating hashed features with nPrint,
4. Applying PCA-based dimensionality reduction,
5. Training and evaluating several classical ML models.

Several key points emerged:

- **Feature configuration matters a lot.**  
  Even when using the same dataset and similar modeling techniques, small changes in nPrint / nPrintML configuration (headers, payload length, hash
  size, etc.) can dramatically alter performance. The relatively modest balanced accuracies I obtained are consistent with the idea that my
  configuration does not match the official NetML feature pipeline.

- **Dimensionality reduction is essential for practicality.**  
  The original feature matrices are large and dense. PCA reduced them from ~1000 dimensions to just 20, making it feasible to train multiple models
  quickly on a laptop. IncrementalPCA and chunked IO were crucial to fit the transforms without exhausting memory.

- **Model choice interacts with preprocessing.**  
  Linear models (Logistic Regression, SGD) benefited from PCA and standardized inputs. Tree-based models (Random Forest, Gradient Boosting, XGBoost) handled
  the PCA-transformed data well and generally achieved better accuracy and balanced accuracy than linear models. However, because the feature extraction
  itself was suboptimal, even the best model did not reach the high numbers reported in the official NetML benchmarks.

- **Easy vs. hard labels.**  
  The easy binary task (malware vs benign) is simpler and generally yields higher accuracy than the hard malware-family task. The hard task suffers from
  class imbalance and substantial overlap between families, resulting in much lower macro-F1 scores.

## 8.2 Future work

For a future version of this project, the most important next steps would be:

1. Match the nPrintML feature configuration (n-gram size, hash length, headers, payload length, and padding rules) to the official settings.
2. Incorporate other AutoML tools to search a larger model space with cross-validation.
3. Explore sequence-based models (e.g., CNNs or Transformers over packet byte sequences) as an alternative to static hashed feature vectors.
4. Explore hyper-parameter settings of the models used. 
