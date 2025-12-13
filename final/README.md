# netML Malware Classification
This project constructs a reproducible preprocessing and modeling pipeline using the netML malware pcapML trace and nPrint features. The project involves dataset analysis
and cleanup, along with an incremental dimensionality reduction to manage a huge dataset, and eventually analyzing multiple ML models to fit both **easy** and **hard**
classes in the dataset.

In addition, the project aims to reproduce the results (and improvize) using netML Autogluon on the above pipeline from ```pcapML``` benchmarks (link: https://nprint.github.io/benchmarks/malware_detection/netml_malware.html).

# Data
The project uses many data files (originally provided and intermediately generated). The directory structure is as follows:

```
data/
  traffic.pcapng.gz
  traffic.pcapng
  traffic_flows/
  flow_features/
  labels_easy.txt
  labels_hard.txt
  all_flow_features_easy.csv
  all_flow_features_hard.csv
  easy_pca_15_components.csv
  hard_pca_15_components.csv
```

## Files originally provided
```
1. traffic.pcapng.gz
```

## Files generated
```
1. traffic.pcapng
2. traffic_flows/
3. flow_features/
4. lables_easy.txt
5. lables_hard.txt
6. all_flow_features_easy.csv
7. all_flow_features_hard.csv
8. easy_pca_15_components.csv
9. hard_pca_15_components.csv
```
## Note
The github repository doesn't provide ```traffic.pcapng.gz``` due to its huge size. For a successful run, this file has to be placed under ```data/```.

# Run

1. ```jupyter notebook```
2. Run the notebook (main.ipynb) from the start.

# Results
1. **RandomForest**, **LogReg**, and **SGD_logreg** achieve >99% balanced accuracy on **easy** class, while 70%, 60%, and 55% on **hard** class.
2. **SVM_rbf** achieves the higest balanced accuracy (~77%) on **hard** class.
3. **netML Autogluon** achieves >99% on **easy**, and ~75% on **hard**.
