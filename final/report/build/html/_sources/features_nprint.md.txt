# 3. Feature Generation with nPrint

The next step is to convert each per-flow PCAP into a fixed-length feature vector
using **nPrint**. These features are essentially hashed summaries of
packet headers and payload bytes.

## 3.1 Per-flow feature extraction

For each per-flow PCAP, I run a command of the form:

```bash
nprint -P <pcap file> -W <features.csv> -4 -6 -t -u -p 20
```
Where:

-P specifies the input PCAP,

-W selects a CSV output file,

-4 includes IPv4 headers,

-6 includes IPV6 headers,

-t includes TCP headers,

-u includes UDP headers,

-p 20 includes 20 bytes of payload.

This yields one CSV per flow, each with a single row of hashed feature values.

Essentially, this is automated in a loop:
```python
import subprocess
from pathlib import Path

flows_dir = Path("./data/traffic_flows")
features_dir = Path("./data/flow_features")
features_dir.mkdir(exist_ok=True)

pcaps = sorted(flows_dir.glob("*.pcap"))

def run_nprint(pcap_path):
    base = pcap_path.stem
    out_csv = features_dir / f"{base}.csv"

    if out_csv.exists():
        return out_csv

    cmd = [
        "nprint",
        "-P", str(pcap_path),
        "-W", str(out_csv),
        "-4", "-6", "-t", "-u",
        "-p", "20"
    ]
    subprocess.run(cmd, check=True)
    return out_csv

for i, p in enumerate(pcaps, 1):
    run_nprint(p)

    # Progress indicator
    if i % 100000 == 0:
        print(f"Processed {i} flows...")

print("Features for all flows in:", features_dir)
```

A new directory ```flow_features``` is created, under which all CSVs are populated.

## Merging per-flow CSVs with labels
This handles dataset creation for training. All CSVs are merged and replicated in 2 unified CSVs (one for **easy** and one for **hard** class).
I create a dictionary by first iterating over the flow labels and then merge all the CSVs by streaming them in the output files (streaming because of memory limitation).

First I generate a map of the labels.
```python
import pandas as pd

easy_df = pd.read_csv("./data/labels_easy.txt", header=None, names=["sample_id", "easy_label"])
hard_df = pd.read_csv("./data/labels_hard.txt", header=None, names=["sample_id", "hard_label"])

easy_df["sample_id"] = easy_df["sample_id"].astype(str)
hard_df["sample_id"] = hard_df["sample_id"].astype(str)

easy_map = dict(zip(easy_df["sample_id"], easy_df["easy_label"]))
hard_map = dict(zip(hard_df["sample_id"], hard_df["hard_label"]))

print("Easy labels:", len(easy_map))
print("Hard labels:", len(hard_map))
```

Then, I generate a unified dataset for each class.

```python
import pandas as pd

easy_df = pd.read_csv("./data/labels_easy.txt", header=None, names=["sample_id", "easy_label"])
hard_df = pd.read_csv("./data/labels_hard.txt", header=None, names=["sample_id", "hard_label"])

easy_df["sample_id"] = easy_df["sample_id"].astype(str)
hard_df["sample_id"] = hard_df["sample_id"].astype(str)

easy_map = dict(zip(easy_df["sample_id"], easy_df["easy_label"]))
hard_map = dict(zip(hard_df["sample_id"], hard_df["hard_label"]))

print("Easy labels:", len(easy_map))
print("Hard labels:", len(hard_map))
```

Eventually, the ```data/``` wounds up with 2 more files: ```all_flow_features_easy.csv``` and ```all_flow_features_hard.csv```.
