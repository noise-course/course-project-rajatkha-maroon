# Project Title

## Project Participants

Name: Rajat Khandelwal

CnetID: rajatkha

Project role: Individual

## Project Description

The problem statement is classifying network flows as malicious or benign based on packet-level information. This problem is particularly important because accurate flow classification enables early detection of anomalies such as DDoS, port scanning, or data exfiltration. By analyzing packet-level features, systems can identify malicious behaviors that bypass traditional signature-based firewalls. This improves real time threat response, secure sensitive data, and maintain critical infrastructure.

The project directy relates to the theme of the course since training on packet traces or flow semantics can help an ML model learn certain behavioral patterns which can be leveraged to detect anamolies. The goal of the project is to reproduce the leaderboard results and hopefully improve upon them by experimenting with different models, and feature-aggregation strategies.

Related work: 
1. Jordan Holland, Paul Schmitt, Nick Feamster, and Prateek Mittal. 2021. New Directions in Automated Traffic Analysis. In Proceedings of the 2021 ACM SIGSAC Conference on Computer and Communications Security (CCS '21). Association for Computing Machinery, New York, NY, USA, 3366–3383. https://doi-org.proxy.uchicago.edu/10.1145/3460120.3484758

## Data

I intend to use the ML/Net malware dataset provided in the leaderboard: https://nprint.github.io/benchmarks/malware_detection/netml_malware.html

## Deliverables

For the final project, I will submit a clean, end-to-end Jupyter notebook that can be executed using “restart kernel and run all” with no manual intervention. The notebook will include all code for converting PCAP files into feature vectors, preprocessing, training, evaluation, and visualizations. 

In addition, I will submit a well-structured written project report formatted in Sphinx, documenting the problem, methodology, dataset, results, and analysis, along with key snippets of the code. 

