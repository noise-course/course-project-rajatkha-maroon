# 2. Dataset and flow extraction

The starting point is the **NetML Malware** dataset, which ships as a single pcapML / pcapng file (e.g. `traffic.pcapng`). Internally, this file encodes many flow-level samples along with metadata.

Each sample has:

- a set of packets (raw bytes for one flow),
- a metadata string that encodes **two labels**:
  - an **easy** binary label (`benign` or `malware`);
  - a **hard** label (malware family name plus benign).

Rather than implementing my own flow-timeout logic, I reuse the flow
boundaries provided by pcapML. This helps keep the pipeline closer to the
intended NetML setup.

I use the `pcalML` utility to build separate **pcap** files across flows.
I use the `pcapml_fe` Python library to iterate over samples and associate the labels.

Furthermore, on close inspection, I found out that the dataset has ambiguous names for benign label. Sometimes the label 'bengin' comes up, and sometimes 'benign7' comes up. I normalize all of them to 'benign'.

```python
# Build flows
!pcapml -M data/traffic.pcapng -O data/traffic_flows/

def normalize_easy_label(lbl: str) -> str:
    lbl = lbl.strip().lower()
    if lbl in ["benign", "bengin", "benign7"]: 
        return "benign"
    elif lbl == "malware":
        return "malware"
    else:
        return lbl 

def normalize_hard_label(lbl: str) -> str:
    lbl = lbl.strip().lower()
    if lbl in ["benign", "benign7", "bengin"]:
        return "benign"
    return lbl

import os
import struct
import pcapml_fe
import re

dataset = "./data/traffic.pcapng"
labels_easy_file = "./data/labels_easy.txt"
labels_hard_file = "./data/labels_hard.txt"

def parse_labels(meta_str):
    s = str(meta_str).strip()
    parts = re.split(r"[,_]", s)
    easy = parts[0]
    hard = parts[1]
    
    return easy, hard


with open(labels_easy_file, "w") as le, open(labels_hard_file, "w") as lh:
    for i, sample in enumerate(pcapml_fe.sampler(dataset)):
        fname = f"{sample.sid}" # Sample ID

        easy, hard = parse_labels(sample.metadata)
        easy = normalize_easy_label(easy)
        hard = normalize_hard_label(hard)

        # Write labels into the file
        le.write(f"{fname},{easy}\n")
        lh.write(f"{fname},{hard}\n")

        # Progress indicator
        if (i + 1) % 100000 == 0:
            print(f"Wrote {i+1} labels...")

print("Easy labels in:", labels_easy_file)
print("Hard labels in:", labels_hard_file)
```

After this stage, the data looks like this:
```
data/
	traffic_flows/
		10000002068136406352_malware_emotet.pcap
		10000026091954898561_bengin_benign.pcap
		...
		...
	labels_easy.txt
	labels_hard.txt
```
Each line of labels_easy/hard.txt is ```sampleId,label```.
