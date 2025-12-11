# 4. Dimensionality reduction with PCA

The merged feature matrices are large: 400k flows and roughly a thousand feature dimensions per flow. This raises two issues:

1. Memory usage: loading the full dense matrix into RAM is heavy.  
2. Redundancy: many nPrint feature dimensions are correlated.

To address both, I use **Principal Component Analysis (PCA)** with **incremental**
updates to reduce dimensionality before model training.

## 4.1. Scree plot on a sample
To decide how many principal components to retain, I first examine the variance captured by the top components on a random sample of flows.
Here is a typical workflow for the easy task:

1. Create a sample of 50k flows
2. Calculate the explained variance and make scree plot

### 4.1.1. Sample of 50k flows
The below code represents the sampling. Since the dataset is large, I stream chunks of it (in 20k flows), and select a sub-sample. Ultimately, I stop when I have 50k
or all the chunks are streamed.

```python
import pandas as pd

EASY_CSV = "all_flow_features_easy.csv"
CHUNKSIZE = 20000
TARGET_SAMPLE = 50000

sample_frames = []
total_collected = 0

for chunk in pd.read_csv(EASY_CSV, chunksize=CHUNKSIZE):
    X = chunk.drop(columns=["sample_id", "easy_label"], errors='ignore')
    X = X.select_dtypes(include=["number"])

    if X.shape[0] == 0:
        continue

    need = TARGET_SAMPLE - total_collected
    if need <= 0:
        break

    k = min(len(X), max(1, need // 2))  # grab up to half of remaining target (just a scheme)
    sampled = X.sample(n=k, random_state=42)
    sample_frames.append(sampled)
    total_collected += len(sampled)

    print("So far collected:", total_collected)

easy_sample = pd.concat(sample_frames, ignore_index=True)
print("Collected total:", len(easy_sample))

# Remove overshooting
if len(easy_sample) > TARGET_SAMPLE:
    easy_sample = easy_sample.sample(TARGET_SAMPLE, random_state=42)

print("Final sample shape:", easy_sample.shape)
```

### 4.1.1. Explained variance and scree plot
This part is straightforward. I calculate the explained variance and plot.
```python
from sklearn.preprocessing import StandardScaler
from sklearn.decomposition import PCA
import matplotlib.pyplot as plt

scaler = StandardScaler()
easy_scaled = scaler.fit_transform(easy_sample)

MAX_COMPONENTS = 200

pca = PCA(n_components=MAX_COMPONENTS)
pca.fit(easy_scaled)

explained = pca.explained_variance_ratio_
cum = explained.cumsum()

plt.plot(explained, marker="o")
plt.title("Scree plot")
plt.xlabel("No. of components")
plt.ylabel("Variance ratio")
plt.grid(True)
plt.show()
```
From this plot, I observe that the first ~20 components capture a large fraction of the variance, and
the curve begins to flatten afterward. Hence, for simplicity and efficiency, I fix:

Number of components = 20 for both easy and hard classes.

## 4.2. Incremental PCA on the full dataset
Instead of loading all rows at once (limited memory), I use chunked reading and ```IncrementalPCA```.

Below code is doing it for **easy** class. Similarly, its done for **hard**.

```python
import pandas as pd
import numpy as np

EASY_CSV = "all_flow_features_easy.csv"
OUT_EASY = "easy_pca_20_components.csv"
CHUNKSIZE = 20000
N_COMPONENTS = 20 

first_write = True
for chunk in pd.read_csv(EASY_CSV, chunksize=CHUNKSIZE):
    sample_ids = chunk["sample_id"].reset_index(drop=True)
    labels = chunk["easy_label"].reset_index(drop=True)

    # build feature matrix
    X = chunk.drop(columns=["sample_id", "easy_label"], errors="ignore")
    X = X.select_dtypes(include=[np.number])

    Xs = scaler_easy.transform(X)
    Xp = pca_easy.transform(Xs)

    df_out = pd.DataFrame(Xp, columns=[f"pc{i}" for i in range(N_COMPONENTS)])
    df_out["sample_id"] = sample_ids
    df_out["easy_label"] = labels

    df_out.to_csv(
        OUT_EASY,
        index=False,
        mode="w" if first_write else "a",
        header=first_write,
    )
    first_write = False

print("Easy PCA fit complete", OUT_EASY)
```
Now, we have two more CSVs (PCA features) for easy and hard classes under ```data```: ```easy_pca_20_components.csv``` and ```hard_pca_20_components.csv```.
