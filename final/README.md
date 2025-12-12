# netML Malware Classification
This project constructs a reproducible preprocessing and modeling pipeline using the netML malware pcapML trace and nPrint features. The project involves dataset analysis
and cleanup, along with an incremental dimensionality reduction to manage a huge dataset, and eventually analyzing multiple ML models to fit both **easy** and **hard**
classes in the dataset.

In addition, the project reproduces the results from ```pcapML``` benchmarks (link: https://nprint.github.io/benchmarks/malware_detection/netml_malware.html).

# Data
The project uses many data files (originally provided and intermediately generated). The directory structure is as follows:

```
data/
  traffic_flows/
  flow_features/
  traffic.pcapng.gz
  labels_easy.txt
  labels_hard.txt
  all_flow_features_easy.csv
  all_flow_features_hard.csv
  easy_pca_20_components.csv
  hard_pca_20_components.csv
```

## Files originally provided
```
1. traffic.pcapng.gz
```

## Files generated
```
1. traffic_flows/
2. flow_features/
3. lables_easy.txt
4. lables_hard.txt
5. all_flow_features_easy.csv
6. all_flow_features_hard.csv
7. easy_pca_20_components.csv
8. hard_pca_20_components.csv
```
## Note
The github repository doesn't provide ```traffic.pcapng.gz``` due to its huge size. For a successful run, this file has to be placed under ```data/```.

# Run

1. Create a python3.8 virtual environment.
2. ```jupyter notebook main.ipynb```
3. Run the notebook from the start.

# Results
1. **RandomForest**, **LogReg**, and **SGD_logreg** achieve >99% balanced accuracy on **easy** class, while 70%, 60%, and 55% on **hard** class.
2. Placeholder for reproduction
