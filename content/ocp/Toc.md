---
title: Openshift TOC
weight: 3
date: 2025-08-24
description: Openshift TOC
---

# OpenShift-customized-Training
## Prerequisites 
 - Aws/Azure  Account
 - Resources used
   - 3 LoadBalancers
   - Route 53
   - AWS s3 bucket
   - 1 VPC
   - 5 Ip addresses
   - NAT gateways
   - 8 VMS with 4 cpu and 8 GB of RAM each 

  **Duration:** 


  
**OpenShift Container Platform architecture**

- Overview of Red Hat Enterprise Linux CoreOS (RHCOS)

- Crio OverView

- Podman Overview

- Overview of journactl



Hands on Lab
 - Crio
 - Podman
 - Journalctl

**Installation and update**
 -  Understand the underline infrstracture/resources requirements.
 - Know Quay.io
 - Know Redhat Registry
 -  know ignition files
 -  Installation with IPI
 -  Installation with user-provisioned infrastructure
 -  Installation on vmware(optional)
 -  Multi tenant Openshift Installation
 -  Configure Network Policy
 -  Installation with customized network plugins
 - **Troubleshooting installation issues**
    - Gathering logs from a failed installation
    - Manually gathering logs with SSH access to your host(s)
    - Manually gathering logs without SSH access to your host(s)
    - Getting debug information from the installation program

**Post_installation_configuration**
This task will take most of the time 

- Configuration of Authentication with Htpasswd
- Configuration of Authentication with Azure AD
- Remove the default virtual admin user (kubeadmin)
- Secure Api with ssl certificate
- Secure Route with Route
- Setting the Ingress Controller
- Restricting the API server to private
- Configuration Default Quota project template
- Configure default limits
- Restrict user for LoadBalancer service.
- Configure Alert Manager
- Updating the global cluster pull secret
- Configure Autoscaling for nodes

- Create infrastracture nodes
- Move all infr related services to infra nodes
 - OpenShift Internal Registry
 - Router pods
 - Monitoring pods
 - Logging pods


**OpenShift Backup and DR**
 - Installation and Configuration of Kasten/Velero
 - Setup the backup of etcd 
 - Recovering from the etcd backup
 


 **Post-installation node tasks**
 - Adding RHEL compute nodes to a cluster if needed
 - Configuring Machine health checks
 - Limitations when deploying machine health checks
 - Node host best practices
 - Configure different type of profile
 - Updating ssh keys for master and worker nodes


 **Post-installation network configuration**
  - Enabling the cluster-wide proxy
  - Configuring ingress cluster traffic
  - Configuring network policy
  - Configuring multitenant isolation by using network policy


**Post-installation storage configuration**
- Dynamic provisioning
- Defining a storage class
- Using Azure file for RWX
- Installation and configuration cephcluster with rook operator to achieve below:
 - Block storage
 - File storage
 - Object Storage

**Know OpenShift Internal Registry**
- Configuring additional trust stores for image registry access
- Configuring storage credentials for the Image Registry Operator


**OpenShift Scc**
- Understanding default scc
- Creating and user custom scc


**Pod Scheduling***
- Default scheduling
 - Infrastructure Topological Levels
 - Affinity
 - Anti Affinity
- Advanced scheduling
  - Pod Affinity and Anti-affinity
  - Node Affinity
  - Node Selectors
  - Taints and Tolerations
- Custom scheduling
 - Deploying the Scheduler


***Troubleshoot***

- Pod related issues
  - Router/Registry Not deploying to correct node
  - Registry not showing contents of NFS mount (persistent volume)
  - Hosts Can No Longer Resolve Each Other During Anisble Install
  - Failure to deploy registry (permissions issues)
  - Application Pod fails to deploy
- Issues with Nodes
  - Nodes being reported as ready, but builds failing
  - Node reporting NotReady
  - Nodes report ready but ETCD health check fails
  - Atomic-openshift-node service fails to start
- Registry issues
  - OpenShift builds fail trying to push image using a wrong IP address for the registry
  - OpenShift build error: failed to push image while using NFS persistent storage
  - Failure to push image to OpenShift’s Registry when backed by shared storage
- Quotas and Limitranges
  - Must make a non-zero request for cpu

- Installation Fails…​

  - Web Console Public URL on a different Port
  - UI Redirecting to the URL of the masters instead of the LB
  - Intermittent Login issues (htpasswd)
  - Build Issues
  - oc new-app runs s2i instead of Docker build
  - Binary Build Fails, citing "BadRequest"

- Issues related to Identity
  - user is unable to login
  - user has two identities
  - How to impersonate user
  - login with service account 




 **Migration from Ocp3 to ocp 4**   


    




 











