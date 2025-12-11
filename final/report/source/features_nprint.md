# 3. Feature Generation with nPrint

The next step is to convert each per-flow PCAP into a fixed-length feature vector
using **nPrint**. These features are essentially hashed summaries of
packet headers and payload bytes.

## 3.1 Per-flow feature extraction

For each per-flow PCAP, I run a command of the form:

```bash
nprint -P 10000002068136406352_malware_emotet.pcap -W 10000002068136406352_malware_emotet.csv -4 -t -u -p 20
```
Where:

-P specifies the input PCAP,

-W selects a CSV output file,

-4 includes IPv4 headers,

-t includes TCP headers,

-u includes UDP headers,

-p 20 includes 20 bytes of payload.

This yields one CSV per flow, each with a single row of hashed feature values.

Essentially, this is automated in a loop:
```python
import subprocess
from pathlib import Path

FLOW_DIR = Path("traffic_flows")
FEATURE_DIR = Path("flow_features")
FEATURE_DIR.mkdir(exist_ok=True)

pcaps = sorted(FLOW_DIR.glob("*.pcap"))
print("Num flows =", len(pcaps))

def run_nprint(pcap_path):
    base = pcap_path.stem
    out_csv = FEATURE_DIR / f"{base}.csv"

    if out_csv.exists():
        return out_csv

    cmd = [
        "nprint",
        "-P", str(pcap_path),
        "-W", str(out_csv),
        "-4", "-t", "-u",
        "-p", "20"
    ]
    subprocess.run(cmd, check=True)
    return out_csv

for i, p in enumerate(pcaps, 1):
    run_nprint(p)
    if i % 50000 == 0:
        print(f"Processed {i} flows...")

print("Done")
```

A new directory ```flow_features``` is created, under which all CSVs are populated.

## Merging per-flow CSVs with labels
This handles dataset creation for traninig. All CSVs are merged and replicated in 2 unified CSVs (one for **easy** and one for **hard** class).
I create a dictionary by first iterating over the flow labels and then merge all the CSVs by streaming them in the output files (streaming because of memory limitation).
```python
from pathlib import Path

FEATURE_DIR = Path("flow_features")
OUT_EASY = Path("all_flow_features_easy.csv")
OUT_HARD = Path("all_flow_features_hard.csv")

csv_files = sorted(FEATURE_DIR.glob("*.csv"))
print("Num CSV files:", len(csv_files))

if not csv_files:
    raise RuntimeError("No CSV files found in flow_features/")


with csv_files[0].open() as f:
    header = f.readline().strip()


with OUT_EASY.open("w") as out_easy, OUT_HARD.open("w") as out_hard:
    # Write combined headers
    out_easy.write(header + ",sample_id,easy_label\n")
    out_hard.write(header + ",sample_id,hard_label\n")

    # Stream one data line per file
    for i, path in enumerate(csv_files, 1):
        fname = path.name
        sample_id = fname.split("_")[0]


        with path.open() as f:
            _ = f.readline() 
            line = f.readline().strip()    # there should be exactly 1 data row

        if not line:
            continue

        # Lookup labels
        sid = str(sample_id)
        easy_label = easy_map.get(sid)
        hard_label = hard_map.get(sid)

        if easy_label is None or hard_label is None:
            continue

        # Write to easy and hard output CSVs
        out_easy.write(f"{line},{sid},{easy_label}\n")
        out_hard.write(f"{line},{sid},{hard_label}\n")

        # Progress track
        if i % 50000 == 0:
            print(f"Merged {i} files...")

print("Done.")
print("Easy CSV:", OUT_EASY)
print("Hard CSV:", OUT_HARD)
```

Eventually, the ```data/``` wounds up with 2 more files: ```all_flow_features_easy.csv``` and ```all_flow_features_hard.csv```.
