---
title: "OpenShift Prerequisites"
weight: 1
date: 2025-08-24
description: "Essential prerequisites for installing OpenShift across different platforms (bare metal, VMware, AWS, Azure, GCP)."
---

## OpenShift Prerequisites  

Before installing OpenShift, ensure the following prerequisites are met for your environment.  

---

### 1. Red Hat Account  
A Red Hat account is required to download the installer, access the pull secret, and manage subscriptions.  
👉 [Create a Red Hat Account](https://console.redhat.com/)

---

### 2. OpenShift Installer  
Download the OpenShift installer appropriate for your target environment (bare metal, VMware, AWS, Azure, or GCP).  
👉 [Download the Installer](https://console.redhat.com/openshift/install)

---

### 3. Pull Secret  
The pull secret is required to access Red Hat’s container registries during installation.  
👉 [Download Pull Secret](https://console.redhat.com/openshift/install/pull-secret)

---

### 4. DNS and Networking  

- Ensure **DNS resolution** is properly configured for the cluster.  
- Required DNS records:  
  - `api.<cluster_name>.<base_domain>` → Load balancer or bootstrap node  
  - `*.apps.<cluster_name>.<base_domain>` → Application load balancer  
- Configure **reverse DNS lookup** for all nodes.  
- Ensure required **firewall ports** are open (API, ingress, node-to-node communication).  
👉 [OpenShift Networking Requirements](https://docs.openshift.com/container-platform/latest/installing/installing_platform_agnostic/installing-platform-agnostic.html#installation-network-user-infra_installing-platform-agnostic)

---

### 5. Infrastructure (VMs or Bare Metal)  

Provision infrastructure based on your target platform:  

#### Bare Metal  
- Minimum of 3 master nodes (control plane)  
- Worker nodes (based on workload requirements)  
- Hardware (per node):  
  - Master: 4 vCPU, 16 GB RAM, 120 GB disk  
  - Worker: 4 vCPU, 8 GB RAM, 120 GB disk  

#### VMware vSphere  
- vCenter access with admin privileges  
- Pre-created VM templates or ability to create VMs  
- Shared datastore and networking  

#### Public Clouds (AWS, Azure, GCP)  
- Proper IAM roles and permissions  
- Region and VPC/subnet prepared  
- Load balancers enabled  
- Storage services (EBS, Azure Disk, GCP Persistent Disk) available  

---

### 6. Cloud CLI Tools  

#### AWS CLI  
👉 [Download AWS CLI](https://aws.amazon.com/cli/)  

#### Azure CLI  
👉 [Download Azure CLI](https://learn.microsoft.com/en-us/cli/azure/)  

#### GCP SDK  
👉 [Download Google Cloud SDK](https://cloud.google.com/sdk/docs/install)  

#### VMware CLI Tools  
- **ovftool** for importing templates  
- vSphere client access  

---

### 7. Certificates and Security  

- Configure TLS certificates (self-signed or custom CA if required).  
- Ensure SSH key pair is generated for cluster node access.  
👉 [Generate SSH Key](https://docs.openshift.com/container-platform/latest/installing/installing-preparing.html#ssh-agent-using_installing-preparing)  

---

### 8. Storage Requirements  

- **Infrastructure storage** for etcd and control plane.  
- **Persistent storage** for applications (NFS, Ceph, GlusterFS, EBS, Azure Disk, GCP PD, VMware vSAN).  
👉 [Storage Requirements](https://docs.openshift.com/container-platform/latest/storage/understanding-persistent-storage.html)  

---

### 9. Operating System Requirements  

- Supported OS for nodes:  
  - Red Hat Enterprise Linux CoreOS (RHCOS) – default  
  - Red Hat Enterprise Linux (RHEL) with prerequisites installed  
- Ensure subscription is attached for RHEL nodes.  

---

### 10. Additional Tools (Optional but Recommended)  

- `oc` CLI (OpenShift client) 👉 [Download OC CLI](https://console.redhat.com/openshift/downloads)  
- `kubectl` CLI (for Kubernetes-level operations)  
- Infrastructure automation tools (Terraform, Ansible) for IaC deployments  

---

✅ With these prerequisites completed, you are ready to begin the **OpenShift installation** on your target platform.  

