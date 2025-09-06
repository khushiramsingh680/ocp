

+++
title = "OpenShift with CodeReady Containers (CRC)"
weight = 3
+++

## Introduction

CodeReady Containers (CRC) provides a minimal OpenShift 4 cluster for development and testing purposes on a local machine. It is ideal for learning, experimenting with OpenShift features, and developing operators or containerized applications.

---

## Key Features

- Single-node OpenShift 4 cluster
- Suitable for laptops and desktops
- Includes OpenShift CLI (`oc`) and Kubernetes CLI (`kubectl`)
- Easy setup with pre-built VM images

---

## System Requirements

| Component       | Requirement                             |
|----------------|------------------------------------------|
| OS              | Linux, macOS, or Windows 10/11          |
| RAM             | 9 GB minimum (12+ GB recommended)       |
| CPU             | 4 cores minimum                         |
| Disk Space      | 35 GB free disk space                   |
| Virtualization  | KVM (Linux), Hyper-V (Windows), HyperKit (macOS) |

---

## Installation Steps

### 1. **Download CRC**

Go to the official Red Hat CRC page:  
👉 [https://developers.redhat.com/products/codeready-containers](https://developers.redhat.com/products/codeready-containers)

Choose the appropriate version for your OS.

---

### 2. **Extract and Install**

```bash
tar -xvf crc-linux-amd64.tar.xz
sudo mv crc-linux-*-amd64/crc /usr/local/bin/
```
## 3 Setup CRC 
> This prepares the host system (adds DNS settings, creates required users/groups, etc.).
```sh
Setup CRC
```
## 4. Start CRC
```sh
crc start
```

## 5. Access OpenShift
```
    OpenShift Console: https://console-openshift-console.apps-crc.testing

    Default credentials:

        Username: developer

        Password: developer
```