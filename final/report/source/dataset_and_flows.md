# 2. Dataset and flow extraction

## 2.1 netML malware pcapML trace
The starting point is the **NetML Malware** dataset, which ships as a single
pcapML / pcapng file (e.g. `traffic.pcapng`). Internally, this file encodes
many flow-level samples along with metadata.

Each sample has:

- a set of packets (raw bytes for one flow),
- a metadata string that encodes **two labels**:
  - an **easy** binary label (`benign` or `malware`);
  - a **hard** label (malware family name plus benign).

Rather than implementing my own flow-timeout logic, I **reuse the flow
boundaries provided by pcapML**. This helps keep the pipeline closer to the
intended NetML setup.

I use the `pcalML` utility to build separate **pcap** files across flows.
I use the `pcapml_fe` Python library to iterate over samples and fetch the labels.

```python
# Build flows
!pcapml -M traffic.pcapng -O traffic_flows/

import os
import struct
import pcapml_fe
import re

pcapml_path = "./traffic.pcapng"
labels_easy_path = "./labels_easy.txt"
labels_hard_path = "./labels_hard.txt"

def parse_labels(meta_str):
    s = str(meta_str).strip()
    parts = re.split(r"[,_]", s)
    easy = parts[0]
    hard = parts[1]
    
    return easy, hard

# Build labels
with open(labels_easy_path, "w") as le, open(labels_hard_path, "w") as lh:
    for i, sample in enumerate(pcapml_fe.sampler(pcapml_path)):
        fname = f"{sample.sid}"

        easy, hard = parse_labels(sample.metadata)

        # Write labels into the file
        le.write(f"{fname},{easy}\n")
        lh.write(f"{fname},{hard}\n")

        # Progress indicator
        if (i + 1) % 100000 == 0:
            print(f"Wrote {i+1} flow pcaps...")

print("Easy labels in:", labels_easy_path)
print("Hard labels in:", labels_hard_path)
...

rajat
