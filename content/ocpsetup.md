---
title: Openshift Installation and Upgradation 
weight: 3
date: 2017-01-05

---
### Openshift Cluster Installation ,Upgradation and Management.

### Prequisites:
-  Login to [AWS Cloud ](https://aws.amazon.com/console/)
-  Create a Virtual Machine
-  Create a admin role and assign to the ec2 instance.

-  Create your account on [Cloud.redhat.com](https://console.redhat.com/)
![](/ocp01.png)
-  Download Installer and Pull secret 
    ![](/ocp02.png)

- Follow the [Redhat document](https://docs.openshift.com/container-platform/4.10/installing/installing_aws/installing-aws-default.html) for Installation 

- Configure AWS Account using [link](https://docs.openshift.com/container-platform/4.8/installing/installing_aws/installing-aws-account.html#installing-aws-account)
-  Get Domain from [Freenom.com](https://www.freenom.com/en/index.html?lang=en)
- Know AWS account [Limit ](https://docs.openshift.com/container-platform/4.8/installing/installing_aws/installing-aws-account.html#installing-aws-account)

- Create one Ec2 instance
- Configure [AWS Cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) with Admin access on newly created ec2 instance.
- Make a test using below command

  ```t
  aws s3 ls
  ```

-  List the OpenShift Container Platform web console route:

```t
oc get routes -n openshift-console | grep 'console-openshift'
```
-  Login to Opeshift Cluster.

```t
oc login -u <username> -p <Password>
```
-  Login to Openshift using kubeconfig
-  Login to OpenShift using Token

```t
oc login --token=<token> --server=<CLuster URL>
```


-  Verifying the Health of OpenShift Nodes
```t
oc get nodes
```
-  Check the cpu and memory usage of each node
```t
oc adm top nodes
```
-  Check more information about a node

```t
oc describe node my-node-name
```

-  Check Openshift cluster Version
```t
oc get clusterversion
oc describe clusterversion
```

-  Check Cluster Operator 
```t
oc get co
```
-  Displaying the Logs of OpenShift Nodes
```t
oc adm node-logs -u crio my-node-name
```
-  Check the logs for kubelet Service 
```t
oc adm node-logs -u kubelet my-node-name
```
-  check all type of logs for a node
```t
oc adm node-logs my-node-name
 ```
-  Login to Cluster node using below command and check kubelet service
```t
oc debug node/my-node-name
systemctl is-active kubelet
```
-  Display the currently authenticated user
```t
oc whoami
```
-  Check the openshift URL
```t
oc whoami  --show-console

```
-  Check openshift Api using below command
```t
oc whoami  --show-server 
```
-  Check the token used by user

```t
oc whoami  --show-token
```
### Debugging Openshift  nodes with crictl

https://kubernetes.io/docs/tasks/debug/debug-cluster/crictl/

# Openshift 4 installation on AWS

## Description

This document describes the installation of OCP4 on the AWS

## RBAC required to manage the installation

Be sure to be able to access to :

- the dedicated AWS environment as Admin for that cluster.
- the Redhat Account.
- to Azure account as Admin to create service Principles.
- Bastion environment.

## Informations about DNS

There should be existing Public Hosted Zone in Route53. In case of stratus, we were having "stratus.soprasteria.com" as Hosted Zone.

## Steps To be Performed Prior to Installation

**Create a SSH key**

For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, you must provide an SSH key that your ssh-agent process uses to the installer. You can use this key to access the bootstrap machine in a public cluster to troubleshoot installation issues.


## Steps to Generate SSH Key

- Generate the ssh-key through below command.
   ```
   ssh-keygen
   ```
- Start the ssh-agent process as a background task
```
eval "$(ssh-agent -s)"
```
- Add SSH private key to the ssh-agent 
```
ssh-add /root/.ssh/id_rsa
```
## Example of install-config.yaml
```
[root@ip-172-31-32-217 ec2-user]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:vr+aWqjvwNJCkqS7ZSdZWzJjt0N1uSAqukjilRd4I/I root@ip-172-31-32-217.eu-west-3.compute.internal
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|           .     |
| .    . o o      |
|o.  .. o o .     |
|+.ooB++ S .      |
| =oB+Oo+         |
|+.BE*.+ o        |
|==.=.o o o       |
|+.  .o+.+oo.     |
+----[SHA256]-----+
[root@ip-172-31-32-217 ec2-user]# eval "$(ssh-agent -s)"
Agent pid 13802
[root@ip-172-31-32-217 ec2-user]# ssh-add /root/.ssh/id_rsa
Identity added: /root/.ssh/id_rsa (root@ip-172-31-32-217.eu-west-3.compute.internal)
```
## Obtaining the installation program

- Access the Infrastructure Provider page on the OpenShift Cluster Manager site and Login to the Redhat Account.
- Select your infrastructure provider.
- Navigate to the page for your installation type, download the installation program for your operating system, and place the    file in the directory where you will store the installation configuration files.
 ```t
 wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
 ``` 
- Extract the installation program

```t
tar xvf openshift-install-linux.tar.gz
```

- Also download the pull secret from the Red Hat OpenShift Cluster Manager.

## Deploying the cluster

- Login to the Bastion server.

- Run the below command to generate the Installation Config file.

```t
./openshift-install create install-config --dir install-new --log-level=debug
```

**NOTE :- As soon as you run the above command, it will ask for basic information like you need to select the Hosted zone, name of the cluster, Zone where you want your VM's to run, ssh key and Pull secret.**


This will create the Install config file. Example for the Installation Config file shown below.


```yaml
apiVersion: v1
data:
  install-config: |
    apiVersion: v1
    baseDomain: <domian>
    compute:
    - architecture: amd64
      hyperthreading: Enabled
      name: worker
      platform: {}
      replicas: 3
    controlPlane:
      architecture: amd64
      hyperthreading: Enabled
      name: master
      platform: {}
      replicas: 3
    metadata:
      creationTimestamp: null
      name: newname
    networking:
      clusterNetwork:
      - cidr: 10.128.0.0/14
        hostPrefix: 23
      machineNetwork:
      - cidr: 10.0.0.0/16
      networkType: OpenShiftSDN
      serviceNetwork:
      - 172.30.0.0/16
    platform:
      aws:
        region: eu-west-3
    publish: External
    pullSecret: ""
    sshKey: |
      ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDeRsMvsXUxzVsnn70zziGkmoCMGwS7A7gswhlvcCoswq1/q3j9yrh91Q8gF0aiKFJmsm45uD1r1DrXc10v+61fmdWcf862YldDBEKF34VN+SJT0wYEy8y3zso8hJ0csi2IB49ywXdw9OZaNk3UrCPbgVhhVZgP4g/thPGNjpJsI10ksuMIE+0Z4cQUbNPR8D1UbOrzPe45ziE1eKYsrRLj0tnQ2Zf8dBRF1NbnmpsfThoQhsQ8e/MGPIaD61BgRr6a/iFHDLYtn1TqyRaUqBrGM1r5hX4Kh6+zI8J1cPmgtNcJhoKvY38npkuKoJYqfhcuvxgKC7k3F43haTwR+5Sz ec2-user@ip-172-31-45-81.eu-west-3.compute.internal
```

**NOTE :- Verify the Install config file to check if you have the desired cluster configuration.**

- Now run the below command to start the Cluster creation Process

```t
./openshift-install create cluster --dir <directory where install-config is present> --log-level=debug
```

Now, wait for the around 40-50 mins for the setup of the cluster where you will get the output similar to below one which will have kube-admin details as well as API cluster link.

```
...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/home/myuser/install_dir/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.mycluster.example.com
INFO Login to the console with user: "kubeadmin", and password: "4vYBz-Ee6gm-ymBZj-Wt5AL"
INFO Time elapsed: 36m22s
```

## Important Links

- Openshift on AWS Installation Link.
https://docs.openshift.com/container-platform/4.7/installing/installing_aws/installing-aws-default.html

- Installation Program Link.
https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/






-   ovnkube resource containers from the nodes (Optional)

```t
for i in $(oc get node | grep master | awk {'print $1'}); do echo $i; oc debug node/${i} -- chroot /host sh -c "mkdir /tmp/ovnlogs && cp /var/log/containers/ovnkube*.log /tmp/ovnlogs"; done
```

-  Test Api server if any issue occurs
```t
oc exec -it apiserver-xxx -c openshift-apiserver -n openshift-apiserver sh

curl -v 10.131.34.32:8443

````

-  Simultaneously we collected tcpdump on all master nodes:
```
$ tcpdump -nn -i any -w /tmp/$(hostname)-$(date +"%Y-%m-%d-%H-%M-%S").pcap
```
 -  if Api server is not reachanble from outside ,login to one of the master node and try below command
 ```
 curl -k https://{IP}:8443/readyz` to all openshift-apiserver pods
 ```
 
 -  How to collect the inspect information for all co
 ```
 https://access.redhat.com/solutions/6995290
 oc adm inspect co --request-timeout=30m
 ```
 
 -  Collect the inspect report for a single project 
 ```t
 oc adm ns inspect ns/openshift-apiserver
 ```
 -  you get the below message once the installation is completed.
 ```t

INFO     export KUBECONFIG=/home/ec2-user/auth/kubeconfig
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.myapp.openshift.gq
INFO Login to the console with user: "kubeadmin", and password: "479aZ-Z2QZy-UdCQy-9d9VN"
```


-  If you have lost all the password like kubeadmin password and kubeconfig file as well you can still run the command.
```t
 cd /etc/kubernetes/static-pod-resources/kube-apiserver-certs/secrets/node-kubeconfigs/
 export KUBECONFIG=$(pwd)/localhost.kubeconfig
 oc whoami

```

## Crictl (Command line tool for managing cri-o based containers and images)

-  pull ubuntu image
```t
sudo crictl pull ubuntu
```
-  List Images
```t
sudo crictl images
```
 -  Delete an image
 ```t
sudo crictl rmi 08d22c0ceb150
 ```
 
-  Print information about specific containers
```t
crictl inspect [container_id1 container_id2 ...]
```
-  Open a specific shell inside a running container
```t
 crictl exec -it [container_id] [sh]
```
-  Pull a specific image from a registry
```t
 crictl pull [image:tag]
```
-  Print and [f]ollow logs of a specific container
```t
 crictl logs -f [container_id]
```
-  Remove one or more images
```t
 crictl rmi [image_id1 image_id2 ...]
```