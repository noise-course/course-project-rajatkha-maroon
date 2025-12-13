# 7. Conclusion, discussion, and future work

## 7.1 Conclusion and discussion

This project explored an end-to-end malware traffic classification pipeline:

1. Starting from the NetML Malware pcapML file
2. Extracting flows and labels
3. Generating features with nPrint
4. Applying PCA-based dimensionality reduction
5. Training and evaluating several classical ML models.

Several key points emerged:

1. **Feature extraction configuration matters**  
  Even when using the same dataset and similar modeling techniques, small changes in nPrint / nPrintML configuration (headers, payload length, hash
  size, etc.) can alter performance. This might be a possible source of inconsistency with my derived balanced accuracies for hard dataset.
2. **Dimensionality reduction is essential for practicality.**  
  The original feature matrices are large and dense. PCA reduced them from 1000 dimensions to just 15, making it feasible to train multiple models
  quickly on a laptop. IncrementalPCA and chunked IO were crucial to fit the transforms without exhausting memory.
3. **Easy vs. hard labels.**  
  The easy binary task (malware vs benign) is simpler and generally yields higher accuracy than the hard malware-family task. The hard task suffers from
  class imbalance and possible overlap between families, hence the lower balanced accuracies.

## 7.2 Future work

For a future version of this project, the most important next steps would be:

1. Match the nPrintML feature configuration to the official settings.
2. Incorporate tools to search a larger model space with cross-validation.
3. Explore sequence-based models (e.g., CNNs or Transformers over packet byte sequences) as an alternative to static hashed feature vectors.
4. Explore hyper-parameter settings of the models used. 
