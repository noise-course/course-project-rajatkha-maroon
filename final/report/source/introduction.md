# 1. Introduction

Modern communication networks continuously generate massive volumes of packet-level data. Automatically classifying this traffic into meaningful application or security categories is essential for network management, intrusion detection, monitoring, and protecting critical infrastructure.

Traditional methods for traffic classification, such as port-based rules or deep packet inspection (DPI), are increasingly ineffective due to encryption, protocol evolution, and deliberate obfuscation. As a result, recent work has moved toward machine-learning–based traffic classification, often using either raw packet bytes or derived statistical features.

In this project, I study malware-versus-benign traffic classification using the NetML Malware dataset and the **nPrintML** framework. nPrintML converts packet captures (pcaps) into fixed-length, feature vectors, which makes it possible to train standard machine learning models on
network flows without writing protocol-specific parsers.

I focus on two related tasks:

* **Easy task**: binary classification of flows as *benign* or *malware*.
* **Hard task**: multi-class classification into malware families
  (plus benign).

The goals of the project are:

* To construct a reproducible preprocessing and modeling pipeline using the netML malware dataset and nPrint features.
* To apply dimensionality reduction (PCA) to make training of such large dataset feasible, and to study how many components are sufficient.
* To compare several classical models on both the easy and hard labels.
* To execute netML Autogluon model and test the results.
