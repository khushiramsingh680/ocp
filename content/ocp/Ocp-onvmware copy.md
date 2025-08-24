---
title: Openshift Installation on Vmware
description: Comprehensive OpenShift guides and documentation.
weight: 2
icon: fab fa-openshift

---
## Description

This document describes the installation of OCP4 on the VMWare infra hosted on premise.

## RBAC required to manage the installation

Be sure to be able to access to :

- the dedicated vCenter https://vcenter01.local.com
- the bastion helper.local.com

## Informations about DNS 
The openshift will be reachable from Internet and from the LAN. It will use a public DNS record.  
A subzone of `local.com.` named  `ocp.local.com` delgated to our team and is managed in an Azure DNS zone to give some freedom and responsivity when we need to manage it.  

## Issues known 

During the installation of the production env, we were hit by the following bug : https://bugzilla.redhat.com/show_bug.cgi?id=1882022 .  
To figure out we applied the following solution: https://access.redhat.com/solutions/5507161 

## Informations required by the installer for the PRODUCTION

| key | value|
| --- | ---  |
| basedomain | local.com |
| metadata.name | ocp4 |
| compute.hyperthreading | Enabled |
| compute.name | worker |
| compute.replicas | 3 |
| compute.platform.vsphere.cpus | 8 |
| compute.platform.vsphere.coresPerSocket | 1 |
| compute.platform.vsphere.memoryMB | 16384 |
| compute.platform.vsphere.osDisk.diskSizeGB | 120 |
| controlPlane.hyperthreading | Enabled |
| controlPlane.name | master |
| controlPlane.replicas | 3 |
| controlPlane.platform.vsphere.cpus | 8 |
| controlPlane.platform.vsphere.coresPerSocket | 1 |
| controlPlane.platform.vsphere.memoryMB | 16384 |
| controlPlane.platform.vsphere.osDisk.diskSizeGB | 120 |
| platform.vsphere.vcenter |  vcenter01.local.com |
| platform.vsphere.username | local.com\administrator |
| platform.vsphere.password  | `find it in the secure place` |
| platform.vsphere.datacenter | 
| platform.vsphere.folder |  |
| platform.vsphere.defaultDatastore | ocp |
| platform.vsphere.network |  ocp |
| platform.vsphere.cluster |  ocp |
| platform.vsphere.api_vip | 192.168.1.100 |
| platform.vsphere.ingress_vip | 192.168.1.101 |
| fips | false |
| pullSecret | find it by going on https://cloud.redhat.com/openshift/create, navigate to the "DataCenter" tab, select "vsphere", and "copy your pull secret |
| sshKey | '' |
| networking.clusterNetwork.cidr |  172.20.0.0/14 |
| networking.clusterNetwork.hostPrefix |  23 |
| networking.clusterNetwork.cidr |  172.20.0.0/14 |
| networking.clusterNetwork.hostPrefix |  23 |
| networking.serviceNetwork | 172.19.0.0/16 |


## Example of install-config.yaml

```yaml
apiVersion: v1
baseDomain: local.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 3
  platform:
    vsphere:
      cpus: 8
      coresPerSocket: 1
      memoryMB: 16384
      osDisk:
        diskSizeGB: 120
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
  platform:
    vsphere:
      cpus: 8
      coresPerSocket: 1
      memoryMB: 16384
      osDisk:
        diskSizeGB: 120
metadata:
  name: ocp4
platform:
  vsphere:
    vcenter: vcenter01.local.com
    username: 
    password: XXXXXXXXXXXXXXXXXXXXXXX
    datacenter:
    folder: 
    defaultDatastore: 
    network: 
    cluster: 
    apiVIP: 
    ingressVIP: 
fips: false
pullSecret: '{"auths":XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX}}}'
sshKey: 
networking:
  clusterNetwork:
    - cidr: 172.20.0.0/14
      hostPrefix: 23
  serviceNetwork:
    - 172.19.0.0/16

```
## The VCenter certificate must be trusted (once !):

```bash 
curl -kSL https://vcenter01.local.com/certs/download.zip -o download.zip
unzip download.zip
sudo cp certs/lin/* /etc/pki/ca-trust/source/anchors
sudo update-ca-trust extract
```

## Prerequisites

- have trusted the certificates on the bastion  
- create an `install-config.yaml` file respecting the values described above
- In the VCenter, manually create the Folder that will be used by the installer.  
For example, for creating the production env folder, go on the `VMs and templates`
- Reserve  2 static IPs that will be used by the ingress and the API
- create the matching Resource Records in the Azure DNS zone 


## Create advanced network configuration

- Connect to the bastion
```bash
 ssh root@<bastion>
 ```

- Create a working directory

```bash 
mkdir -p vsphere/$(date -I)
cd vsphere
```

- copy the install-config.yaml file you created in the working directory
```bash
cp install-config.yaml $(date -I)/
tolerations:
        - effect: NoSchedule 
          key: node-role.kubernetes.io/infra 
          operator: Exists
```

- download the installer program. You can find all archived 4 versions under the following URL: https://mirror.openshift.com/pub/openshift-v4/clients/ocp  
While writting this doc, the version we deployed in other production env is the `4.6.18`.
```bash
vsphere $ wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.6.18/openshift-install-linux-4.6.18.tar.gz
vsphere $ tar xzvf openshift-install-linux-4.6.18.tar.gz
```

- create the manifests file:
``` bash 
./openshift-install create manifests --dir=$(date -I)/
```

- Create a file that is named cluster-network-03-config.yml 
```bash
touch $(date -I)/manifests/cluster-network-03-config.yml 
```

- Add the following content in this file
```yaml
apiVersion: operator.openshift.io/v1
kind: Network
metadata:
  name: cluster
spec:
  clusterNetwork:
  - cidr: 172.20.0.0/14
    hostPrefix: 23
  serviceNetwork:
  - 172.19.0.0/16
  defaultNetwork:
    type: OVNKubernetes
    ovnKubernetesConfig:
      mtu: 1400
      genevePort: 6081
  
```

## Run the installer

- from the same working directory you created during the advanced network config step, run the openshift install tool
```bash
./openshift-install create cluster --dir=./$(date -I) --log-level=info
```

In production env it took around 40 minutes to be run:

```bash
INFO Creating infrastructure resources...
INFO Waiting up to 20m0s for the Kubernetes API at https://api.ocp4.local.com:6443...
INFO API v1.19.0+f173eb4 up
INFO Waiting up to 30m0s for bootstrapping to complete...
INFO Destroying the bootstrap resources...
INFO Waiting up to 40m0s for the cluster at https://api.ocp4.local.com:6443 to initialize...
INFO Waiting up to 10m0s for the openshift-console route to be created...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/ocpadmin/vsphere/2021-03-25/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.ocp4.local.com
INFO Login to the console with user: "kubeadmin", and password: "XXXXXXXXXXXX"
INFO Time elapsed: 38m24s
```

## install the oc client on the bastion (once)

- download and install oc
```bash
curl https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.6.18/openshift-client-linux-4.6.18.tar.gz -o openshift-client-linux-4.6.18.tar.gz
tar xzvf openshift-client-linux-4.6.18.tar.gz
sudo cp oc /usr/local/bin/
sudo cp kubectl /usr/local/bin/
```

- Manage the auto completion
```bash
sudo yum install -y bash-completion
oc completion bash > ~/.kube/completion.bash.inc
printf "
# Kubectl shell completion
source '$HOME/.kube/completion.bash.inc'
" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Registry installation

- temporarily decrease the registry replicas to 1
```bash
oc patch config.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"rolloutStrategy":"Recreate","replicas":1}}'
```

- Define a PVC of `100Gi` in a 'pvc.yaml' file

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: image-registry-storage 
spec:
  accessModes:
  - ReadWriteOnce 
  resources:
    requests:
      storage: 100Gi 
```

- create it in the right namespace
```bash
oc create -f pvc.yaml -n openshift-image-registry  
 ```

 - edit the registry configuration
 ```bash
  oc edit config.imageregistry.operator.openshift.io -o yaml
 ```

 - and modify the spec.storage configuration to match the following settings:

 ```yaml
   storage:
    pvc:
      claim: image-registry-storage 
 ```

 - Wait for the PV has been successfully created and make the registry managed

 ```bash
 oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
 ```

 - Wait for the registry becomes ready

 ```bash
 watch oc get po -l docker-registry=default 
 ```

 - Check the cluster operator became ready in a non-degraded state
 ```bash
  oc get co image-registry
```



- Test the registry

## [External references]( https://docs.openshift.com/container-platform/4.6/installing/installing_vsphere/installing-vsphere-installer-provisioned-customizations.html#installation-installer-provisioned-vsphere-config-yaml_installing-vsphere-installer-provisioned-customizations)


