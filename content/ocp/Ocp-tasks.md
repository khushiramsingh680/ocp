---
title: Openshift 4 Tasks
description: Openshift
weight: 3

---
- [OpenShift Administrators Roles and Responsibilities](#openshift-administrators-roles-and-responsibilities)
- [Updating SSH Keys on OpenShift 4 Nodes via MachineConfig](#updating-ssh-keys-on-openshift-4-nodes-via-machineconfig)
- [Adding an SSL Certificate to Trusted Store in OpenShift Cluster](#adding-an-ssl-certificate-to-trusted-store-in-openshift-cluster)
- [SSL Related Tasks in OpenShift 4 Administration](#ssl-related-tasks-in-openshift-4-administration)
- [Updating the Default SSL Certificate](#updating-the-default-ssl-certificate)
- [Changing Configuration for All OpenShift Nodes](#changing-configuration-for-all-openshift-nodes)
- [Adjusting Kernel Parameters](#adjusting-kernel-parameters)
- [Changing the Default Container Runtime Settings](#changing-the-default-container-runtime-settings)
- [Upgrading OpenShift Cluster](#upgrading-openshift-cluster)
- [Upgrading OpenShift Nodes in Batches](#upgrading-openshift-nodes-in-batches)
- [How to change the no of node unavailable while upgrading](#how-to-change-the-no-of-node-unavailable-while-upgrading)
- [Modifying OpenShift Project Templates](#modifying-openshift-project-templates)
- [Adding Resource Quotas](#adding-resource-quotas)
- [Adding LimitRanges](#adding-limitranges)
- [Exposing the Internal OpenShift Registry](#exposing-the-internal-openshift-registry)
- [Overview of the `openshift-config` Namespace in OpenShift](#overview-of-the-openshift-config-namespace-in-openshift)
##  OpenShift Administrators Roles and Responsibilities.
1. **Capacity Planning**
1. **Proof of concepts**
1. **Cluster Installation and Upgrades**  
   - Set up new clusters, apply regular updates, and perform major upgrades.
    This includes managing Red Hat OpenShift Cluster Manager and leveraging tools like Red Hat Advanced Cluster Management (RHACM) for multi-cluster environments.

2. **User Management and Access Control**  
   - Implement role-based access control (RBAC) policies, manage users and permissions, and integrate authentication providers like LDAP or Active Directory.

3. **Storage Management**  
   - Configure persistent storage using OpenShift Container Storage (OCS) or other storage providers. Manage storage classes, Persistent Volumes (PVs), and Persistent Volume Claims (PVCs) to ensure reliable data storage for applications.

4. **Networking and Ingress Configuration**  
   - Manage networking policies, routes, ingress controllers, and network security configurations. Tasks include setting up cluster networking (SDN or OpenShift SDN), load balancing, and managing multi-cluster networking with RHACM.

5. **Monitoring and Logging**  
   - Use Prometheus and Grafana for monitoring, and EFK/ELK stacks (Elasticsearch, Fluentd, Kibana) for centralized logging. Set up alerting for resource utilization, network traffic, and error logs to proactively manage cluster health.

6. **Security Compliance and Vulnerability Management**  
   - Apply security patches, manage OpenShift's integrated security features, and scan images for vulnerabilities. Enable features like the OpenShift Compliance Operator for automated security checks.

7. **Resource Optimization and Quota Management**  
   - Monitor and optimize resource usage (CPU, memory), and set quotas and limits for namespaces and projects to prevent resource exhaustion and ensure fair resource allocation.

8. **Backup and Disaster Recovery**  
   - Implement backup and restore strategies using tools like Velero for Kubernetes backups, Kasten, or Red Hat-supported backup solutions. DR planning includes setting up automated backups and validating restore procedures regularly.

9. **Operator Lifecycle Management (OLM)**  
   - Install, upgrade, and manage Operators for automating infrastructure components and applications. Ensure Operators are kept up-to-date and compatible with the OpenShift environment.

10. **Support and Troubleshooting**  
    - Regularly troubleshoot performance issues, manage logs, resolve configuration errors, and work with Red Hat Support or internal resources for escalations. Common issues include network latency, failed deployments, or resource constraints impacting applications.

These tasks keep an OpenShift 4 environment secure, high-performing, and aligned with organizational requirements, especially for production-grade clusters.

##  Updating SSH Keys on OpenShift 4 Nodes via MachineConfig

OpenShift 4 uses the Machine Config Operator (MCO) to manage node configurations. To update SSH keys across all master and worker nodes, create a `MachineConfig` resource to apply the new SSH public key.

### Step 1: Generate a New SSH Key Pair (Optional)
If you don’t already have an SSH key pair, generate a new one on your local machine:
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
This command creates two files:
- ~/.ssh/id_rsa (private key)
- ~/.ssh/id_rsa.pub (public key)

### Step 2: Create a MachineConfig YAML File for SSH Key Update
Create a YAML file, e.g., `ssh-key-update.yaml`, with the following content. Replace `<your-ssh-public-key>` with the actual content of your SSH public key.
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: ssh-key-update
  labels:
    machineconfiguration.openshift.io/role: <node-role>  # Replace <node-role> with "worker" or "master"
spec:
  config:
    ignition:
      version: 3.2.0
    passwd:
      users:
        - name: core
          sshAuthorizedKeys:
            - "ssh-rsa <your-ssh-public-key>"
```
- **machineconfiguration.openshift.io/role**: Set this to `worker` to target worker nodes or `master` to target master nodes.
- **sshAuthorizedKeys**: Add your SSH public key here (the content of `id_rsa.pub`).

### Step 3: Apply the MachineConfig
1. Apply the `MachineConfig` to your OCP 4 cluster:
```bash
oc apply -f ssh-key-update.yaml
```
2. Monitor the rollout status to confirm that the update is applied across nodes:
```bash
oc get machineconfigpool
```
Wait until the `UPDATED` status is `True` for the respective `machineconfigpool` (e.g., `worker` or `master`).

### Step 4: Verify the SSH Key Update
Once the rollout is complete, verify that you can SSH into the nodes as the `core` user:
```bash
ssh core@<node-ip>
```
This should authenticate using the updated SSH key.

---

**Important Notes**:
- **Do not manually modify SSH keys** directly on nodes, as the Machine Config Operator will revert changes outside of its control.
- **Role-based MachineConfig**: If you need different keys for master and worker nodes, create separate `MachineConfig` files, each with the appropriate `machineconfiguration.openshift.io/role` label.
- **Automation**: Changes applied with MachineConfig will persist across reboots and node replacements, aligning with OpenShift's immutable infrastructure approach.


##   Adding an SSL Certificate to Trusted Store in OpenShift Cluster

To add an SSL certificate to the trusted store in an OpenShift cluster, follow these steps:

### Step 1: Obtain the SSL Certificate
Make sure you have the SSL certificate file (e.g., `my-certificate.crt`) that you want to add to the trusted store.

### Step 2: Create a ConfigMap or Secret
You can either use a `ConfigMap` or a `Secret` to store the SSL certificate. A `Secret` is recommended for sensitive data.

### Create a Secret
1. **Create a Secret containing the SSL certificate:**
```bash
oc create secret generic my-ssl-cert --from-file=ca.crt=my-certificate.crt -n <namespace>
```
Replace `<namespace>` with the desired namespace where you want to create the secret.

### Create a ConfigMap (Optional)
If you choose to use a `ConfigMap`, you can create it with:
```bash
oc create configmap my-ssl-cert --from-file=my-certificate.crt -n <namespace>
```

### Step 3: Configure the Trusted CA
To configure the OpenShift cluster to trust the SSL certificate, you need to add it to the appropriate configuration.

### For Custom Certificates
1. **Edit the `ClusterIP` service to use the custom CA:**
```bash
oc patch configmap/cluster --type merge -p '{"data":{"trust-ca.crt": "your-certificate-data"}}' -n openshift-config
```
Replace `"your-certificate-data"` with the actual contents of your SSL certificate.

### Step 4: Update the API Server
1. **Edit the API server configuration to include your SSL certificate:**
```bash
oc patch apiserver cluster --type merge --patch '{"spec":{"trustedCA":{"name":"my-ssl-cert"}}}'
```

### Step 5: Restart Affected Pods
To apply the changes, you may need to restart the affected pods, such as the API server or any applications that need to trust the new certificate.

1. **Restart the affected pods:**
```bash
oc rollout restart deployment/<deployment-name> -n <namespace>
```

### Step 6: Verify the Configuration
1. **Verify that the certificate has been added to the trusted store:**
You can check the trusted certificates by accessing a pod and verifying the CA store:
```bash
oc exec -it <pod-name> -- cat /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
```

---

**Important Notes**:
- Always ensure you have the correct permissions to create secrets and modify configurations in the cluster.
- Be cautious when handling sensitive information like SSL certificates.
- Depending on your OpenShift version and setup, some steps may vary slightly.


##  SSL Related Tasks in OpenShift 4 Administration

As an OpenShift 4 administrator, managing SSL certificates is crucial for securing communications. This guide outlines common SSL-related tasks, including adding custom certificates, renewing certificates, and managing the default certificate.

### Table of Contents
1. [Adding a Custom SSL Certificate](#adding-a-custom-ssl-certificate)
2. [Renewing SSL Certificates](#renewing-ssl-certificates)
3. [Updating the Default SSL Certificate](#updating-the-default-ssl-certificate)
4. [Verifying SSL Certificates](#verifying-ssl-certificates)

###  Adding a Custom SSL Certificate

### Step 1: Obtain the SSL Certificate
Ensure you have the SSL certificate files (e.g., `my-certificate.crt` and `my-key.key`).

### Step 2: Create a Secret for the SSL Certificate
1. **Create a Secret containing the SSL certificate and key:**
```bash
oc create secret tls my-tls-secret --cert=my-certificate.crt --key=my-key.key -n <namespace>
```
Replace `<namespace>` with the desired namespace.

### Step 3: Configure Ingress or Routes to Use the Custom SSL Certificate
1. **Edit your Ingress or Route to reference the Secret:**
```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: my-route
  namespace: <namespace>
spec:
  host: myapp.example.com
  to:
    kind: Service
    name: my-service
  tls:
    termination: edge
    certificate: |
      -----BEGIN CERTIFICATE-----
      <your-certificate-content>
      -----END CERTIFICATE-----
    key: |
      -----BEGIN PRIVATE KEY-----
      <your-key-content>
      -----END PRIVATE KEY-----
```

###  Renewing SSL Certificates

### Step 1: Obtain the New SSL Certificate
Acquire the new SSL certificate files when it's time to renew your certificate.

### Step 2: Update the Existing Secret
1. **Update the Secret with the new certificate and key:**
```bash
oc create secret tls my-tls-secret --cert=new-certificate.crt --key=new-key.key -n <namespace> --dry-run=client -o yaml | oc apply -f -
```

### Step 3: Verify the Route
Ensure that the updated SSL certificate is being used:
```bash
oc get route my-route -n <namespace> -o yaml
```

## Updating the Default SSL Certificate

### Step 1: Create or Update a ConfigMap
If you need to update the default certificates for the cluster (e.g., for the API server), create or update a `ConfigMap`:
```bash
oc create configmap custom-ca --from-file=my-certificate.crt -n openshift-config --dry-run=client -o yaml | oc apply -f -
```

### Step 2: Patch the API Server
1. **Patch the API server to use the new custom CA:**
```bash
oc patch apiserver cluster --type merge --patch '{"spec":{"trustedCA":{"name":"custom-ca"}}}'
```

### Verifying SSL Certificates

### Step 1: Verify SSL Certificate in Use
You can verify which SSL certificate is currently in use for a route:
```bash
oc get route my-route -n <namespace> -o jsonpath='{.spec.tls}'
```

### Step 2: Check the Certificate Chain
To check the certificate chain of a service, you can exec into a pod and use tools like `openssl`:
```bash
oc exec -it <pod-name> -- /bin/sh
openssl s_client -connect myapp.example.com:443 -showcerts
```

---

**Important Notes**:
- Ensure you have the necessary permissions to manage secrets and configurations in OpenShift.
- Always back up existing certificates and configurations before making changes.
- Monitor the expiration of certificates and set reminders to renew them in advance.


##  Changing Configuration for All OpenShift Nodes

OpenShift 4 uses the Machine Config Operator to manage the configuration of nodes. To change the configuration for all nodes, follow these steps:

### Step 1: Identify the Configuration Change

Determine what specific configuration you need to change. Common changes include:
- Updating SSH keys
- Modifying systemd services
- Adjusting kernel parameters
- Changing the default container runtime settings

### Step 2: Create a MachineConfig

1. **Create a YAML file for the MachineConfig**. For example, to update the SSH keys, create a file named `update-ssh-keys.yaml` with the following content:
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: update-ssh-keys
  labels:
    machineconfiguration.openshift.io/role: worker # or master
spec:
  config:
    ignition:
      version: 3.2.0
    passwd:
      users:
        - name: core
          sshAuthorizedKeys:
            - "ssh-rsa <your-new-ssh-public-key>"
```
Replace `<your-new-ssh-public-key>` with the actual content of the new SSH public key.

2. **Apply the MachineConfig**:
```bash
oc apply -f update-ssh-keys.yaml
```

### Step 3: Monitor the Status of the Machine Config Pool

After applying the MachineConfig, you should monitor the status of the affected Machine Config Pool to ensure the changes are applied:
```bash
oc get machineconfigpool
```
Wait until the `UPDATED` status is `True` for the respective Machine Config Pool (e.g., `worker` or `master`).

### Step 4: Verify the Configuration Change

1. **SSH into one of the nodes** to verify the configuration change:
```bash
ssh core@<node-ip>
```
2. **Check the updated configuration** as needed, for example, to verify SSH keys:
```bash
cat ~/.ssh/authorized_keys
```

### Step 5: Restart Affected Services or Pods

Some changes might require restarting affected services or pods. To restart the affected pods, you can use:
```bash
oc rollout restart deployment/<deployment-name> -n <namespace>
```

### Important Notes

- **Machine Config Operator**: Be cautious when using the Machine Config Operator, as incorrect configurations can lead to node failures.
- **Backup Existing Configurations**: Always backup your existing configurations before making changes.
- **Node Role**: Make sure to specify the correct node role (master or worker) in the labels of your MachineConfig.
- **Multiple Changes**: If you have multiple changes to apply, consider creating a single MachineConfig that encapsulates all required changes, rather than creating multiple individual MachineConfigs.

---

This guide provides a structured approach to changing configurations for all OpenShift nodes. If you have specific configurations in mind or need further assistance, feel free to ask!


###  Changing Configuration for All OpenShift Nodes

OpenShift 4 uses the Machine Config Operator (MCO) to manage the configuration of nodes. This guide provides steps to modify systemd services, adjust kernel parameters, and change default container runtime settings.

### Modifying Systemd Services

### Step 1: Create a MachineConfig for Systemd Modifications

1. **Create a YAML file for the MachineConfig**. For example, to modify a systemd service, create a file named `modify-systemd.yaml` with the following content:
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: modify-systemd
  labels:
    machineconfiguration.openshift.io/role: worker # or master
spec:
  config:
    ignition:
      version: 3.2.0
    systemd:
      units:
        - name: my-service.service
          enabled: true
          dropins:
            - name: 10-my-custom.conf
              contents: |
                [Service]
                Environment="MY_ENV_VAR=my_value"
```
Replace `my-service.service` and the `Environment` variables with your actual service name and configuration.

### Step 2: Apply the MachineConfig
```bash
oc apply -f modify-systemd.yaml
```

### Step 3: Verify the Change
SSH into the node and check the status of the modified service:
```bash
ssh core@<node-ip>
systemctl status my-service.service
```

## Adjusting Kernel Parameters

### Step 1: Create a MachineConfig for Kernel Parameters

1. **Create a YAML file for the MachineConfig**. For example, to adjust kernel parameters, create a file named `adjust-kernel-params.yaml` with the following content:
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: adjust-kernel-params
  labels:
    machineconfiguration.openshift.io/role: worker # or master
spec:
  config:
    ignition:
      version: 3.2.0
    kernelArguments:
      - "parameter1=value1"
      - "parameter2=value2"
```
Replace `parameter1=value1` and `parameter2=value2` with the actual kernel parameters you need to set.

### Step 2: Apply the MachineConfig
```bash
oc apply -f adjust-kernel-params.yaml
```

### Step 3: Verify the Change
SSH into the node and check the current kernel parameters:
```bash
ssh core@<node-ip>
sysctl -a | grep parameter1
```

##   Changing the Default Container Runtime Settings

### Step 1: Create a MachineConfig for Container Runtime Settings

1. **Create a YAML file for the MachineConfig**. For example, to change container runtime settings, create a file named `change-runtime-settings.yaml` with the following content:
```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: change-runtime-settings
  labels:
    machineconfiguration.openshift.io/role: worker # or master
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
        - path: /etc/crio/crio.conf
          mode: 0644
          owner: root:root
          contents:
            inline: |
              [crio]
              default_runtime = "my-runtime"
              [runtime]
              runtime_type = "my-runtime"
```
Replace `my-runtime` with your actual runtime settings.

### Step 2: Apply the MachineConfig
```bash
oc apply -f change-runtime-settings.yaml
```

### Step 3: Verify the Change
SSH into the node and check the runtime configuration:
```bash
ssh core@<node-ip>
cat /etc/crio/crio.conf
```

### Step 4: Monitor the Machine Config Pool
After applying any of the MachineConfigs, monitor the Machine Config Pool:
```bash
oc get machineconfigpool
```
Wait until the `UPDATED` status is `True` for the respective Machine Config Pool (e.g., `worker` or `master`).

---

**Important Notes**:
- **Machine Config Operator**: Be cautious when using the MCO, as incorrect configurations can lead to node failures.
- **Backup Existing Configurations**: Always backup your existing configurations before making changes.
- **Node Role**: Ensure to specify the correct node role (master or worker) in the labels of your MachineConfig.
- **Service Impact**: Some changes may require restarting the affected services or pods for the changes to take effect.

This guide provides clear instructions on modifying systemd services, adjusting kernel parameters, and changing container runtime settings in OpenShift nodes. If you have specific requirements or additional questions, feel free to ask!


##  Upgrading OpenShift Cluster

Upgrading an OpenShift cluster requires careful planning and execution. This guide outlines the steps for upgrading an OpenShift cluster.

### Prerequisites

1. **Backup Important Data**:
   - Backup your current cluster state, including etcd data and any custom resources.

2. **Review Release Notes**:
   - Check the [OpenShift Release Notes](https://docs.openshift.com/) for any specific upgrade instructions and deprecated features.

3. **Check Compatibility**:
   - Verify that your current version is compatible with the target version.

4. **Update CLI Tools**:
   - Ensure you have the latest version of `oc` and `openshift-install` CLI tools that match the version you're upgrading to.

### Step 1: Prepare for the Upgrade

1. **Check Cluster Health**:
   ```bash
   oc get nodes
   oc get pods --all-namespaces
   oc get clusteroperators
   ```

2. **Drain Nodes**:
   - Start with the master nodes, draining them to prepare for the upgrade.
   ```bash
   oc adm drain <master-node-name> --ignore-daemonsets --force --delete-local-data
   ```

3. **Update Machine Configurations** (if needed):
   - If there are any machine configurations that need updates before the upgrade, apply those changes now.

### Step 2: Upgrade the Cluster

1. **Start the Upgrade Process**:
   - Trigger the upgrade process using the OpenShift CLI.
   ```bash
   oc adm upgrade begin --to-image=<new-openshift-image>
   ```

2. **Monitor the Upgrade**:
   - Check the status of the upgrade process.
   ```bash
   oc get clusterversion
   ```

3. **Allow Upgrade to Complete**:
   - Wait for the upgrade process to complete. Ensure that all cluster operators are reporting as `Available`.

### Step 3: Post-Upgrade Tasks

1. **Uncordon Nodes**:
   - Once the upgrade is complete, uncordon the master nodes and worker nodes.
   ```bash
   oc adm uncordon <master-node-name>
   oc adm uncordon <worker-node-name>
   ```

2. **Check Cluster Health Again**:
   ```bash
   oc get nodes
   oc get pods --all-namespaces
   oc get clusteroperators
   ```

3. **Update Machine Configurations (if necessary)**:
   - If there are additional machine configurations or settings required for the new version, apply those now.

4. **Validate Application Functionality**:
   - Verify that all applications are functioning as expected after the upgrade.

### Step 4: Clean Up

1. **Remove Old Images**:
   - Clean up unused images from the nodes to free up space.
   ```bash
   oc adm prune images
   ```

2. **Monitor Cluster Performance**:
   - Keep an eye on the cluster performance after the upgrade to ensure everything is running smoothly.

### Important Notes

- **Upgrade Path**: Always follow the recommended upgrade path (e.g., from 4.x to 4.y) as specified in the OpenShift documentation.
- **Test Upgrades**: If possible, test the upgrade process in a staging environment before performing it in production.
- **Documentation**: Refer to the official [OpenShift Upgrade Documentation](https://docs.openshift.com/) for detailed instructions specific to your version.

This guide provides a structured approach to upgrading your OpenShift cluster. If you have specific requirements or encounter issues during the upgrade process, feel free to ask for assistance!



##  Upgrading OpenShift Nodes in Batches

When upgrading an OpenShift cluster with multiple nodes, you can upgrade nodes in batches (e.g., three at a time) to minimize downtime and ensure a smooth transition.

### Prerequisites

1. **Backup Your Cluster**: Before proceeding, back up your etcd and any critical application data.
2. **Review Release Notes**: Check the OpenShift release notes for any breaking changes or important notes about the upgrade.
3. **Upgrade Plan**: Decide on the upgrade order for your nodes (typically, master nodes are upgraded first, followed by worker nodes).

### Step 1: Prepare for the Upgrade

1. **Check Cluster Health**:
   ```bash
   oc get nodes
   oc get pods --all-namespaces
   oc get clusteroperators
   ```

2. **Drain the Nodes**:
   - Start with the nodes you plan to upgrade. Drain three nodes at a time.
   ```bash
   for node in node1 node2 node3; do
       oc adm drain $node --ignore-daemonsets --force --delete-local-data
   done
   ```

   Replace `node1`, `node2`, and `node3` with the actual node names.

### Step 2: Upgrade the Nodes

1. **Start the Upgrade Process**:
   - Begin the upgrade for the selected nodes.
   ```bash
   for node in node1 node2 node3; do
       oc adm upgrade begin --to-image=<new-openshift-image> --node=$node
   done
   ```

   Replace `<new-openshift-image>` with the image for the target OpenShift version.

2. **Monitor the Upgrade**:
   - Check the status of the upgrade process for each node.
   ```bash
   oc get clusterversion
   ```

   Wait until the upgrade process completes for all three nodes.

### Step 3: Uncordon the Nodes

1. **Uncordon the Upgraded Nodes**:
   - Once the upgrade is complete, uncordon the nodes to make them schedulable again.
   ```bash
   for node in node1 node2 node3; do
       oc adm uncordon $node
   done
   ```

### Step 4: Repeat for Remaining Nodes

1. **Continue Upgrading**:
   - Repeat the above steps for the next batch of three nodes until all nodes are upgraded.
   ```bash
   # Upgrade next batch
   for node in node4 node5 node6; do
       oc adm drain $node --ignore-daemonsets --force --delete-local-data
       oc adm upgrade begin --to-image=<new-openshift-image> --node=$node
       oc adm uncordon $node
   done
   ```

### Step 5: Post-Upgrade Verification

1. **Check Cluster Health Again**:
   ```bash
   oc get nodes
   oc get pods --all-namespaces
   oc get clusteroperators
   ```

2. **Validate Application Functionality**:
   - Ensure that all applications are functioning as expected after the upgrade.

### Important Notes

- **Plan for Downtime**: Upgrading nodes in batches may cause temporary unavailability for applications running on those nodes. Ensure your applications are resilient and can handle node failures.
- **Monitor Performance**: After upgrading, monitor cluster performance and health metrics to ensure everything is running smoothly.
- **Follow Upgrade Path**: Always adhere to the recommended upgrade paths specified in the OpenShift documentation.

By following this guide, you can effectively upgrade three nodes at a time in your OpenShift cluster. If you have any specific requirements or encounter issues during the upgrade process, feel free to ask for assistance!


##  How to change the no of node unavailable while upgrading 

```bash
oc patch mcp worker --type merge --patch '{"spec": {"maxUnavailable": 2}}'
```
### Verify the maxUnavailable value:
```yaml
oc get mcp worker -o yaml | grep maxUnavailable
```


##  Modifying OpenShift Project Templates

OpenShift provides the ability to create and modify project templates, which define a standard set of resources and configurations that can be instantiated to create new projects or applications quickly. Modifying a project template allows you to customize default behaviors and settings for your applications.

### Key Components of a Project Template
- **Parameters:** Variables that can be customized when instantiating the template.
- **Objects:** The resources that will be created when the template is applied (e.g., DeploymentConfigs, Services, Routes).
- **Labels and Annotations:** Metadata for categorizing and describing resources.

### Viewing Existing Templates
To view the templates available in your OpenShift environment, use:
```bash
oc get templates
```

### Modifying a Template
You can modify an existing template by following these steps:

### 1. Export the Template
To export a template to a file for editing:
```bash
oc get template <template-name> -o yaml > template.yaml
```
Replace `<template-name>` with the name of the template you want to modify.

### 2. Edit the Template
Open the `template.yaml` file in your preferred text editor and make the necessary changes. Key areas to consider modifying:
- **Parameters:** Add, remove, or change parameter definitions.
- **Objects:** Modify or add resource specifications (e.g., change replicas in a DeploymentConfig).
- **Labels/Annotations:** Update or add labels and annotations for better resource management.

### 3. Apply the Modified Template
Once you’ve made your modifications, you can apply the updated template back to OpenShift:
```bash
oc apply -f template.yaml
```

## Adding Resource Quotas
Resource quotas are used to limit the amount of resources that can be consumed in a project. To add a resource quota to your template, include a `ResourceQuota` object in the `objects` section of the template:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: example-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "4"
    limits.memory: "8Gi"
```

## Adding LimitRanges
Limit ranges set default request and limit values for containers in a project. You can add a `LimitRange` object in the `objects` section of the template:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: example-limits
spec:
  limits:
    - default:
        cpu: "500m"
        memory: "1Gi"
      defaultRequest:
        cpu: "250m"
        memory: "512Mi"
      type: Container
```

### Creating a New Template
If you want to create a new project template from scratch, you can use:
```bash
oc create -f <template-file>.yaml
```
Make sure to define parameters, objects, and metadata in the YAML file.

### Example Template Structure
Here’s a simple example structure of a project template in YAML, including resource quotas and limit ranges:

```yaml
apiVersion: template.openshift.io/v1
kind: Template
metadata:
  name: example-template
  labels:
    app: example
parameters:
  - name: APP_NAME
    description: The name of the application
    required: true
objects:
  - apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: example-quota
    spec:
      hard:
        requests.cpu: "4"
        requests.memory: "8Gi"
        limits.cpu: "4"
        limits.memory: "8Gi"
  - apiVersion: v1
    kind: LimitRange
    metadata:
      name: example-limits
    spec:
      limits:
        - default:
            cpu: "500m"
            memory: "1Gi"
          defaultRequest:
            cpu: "250m"
            memory: "512Mi"
          type: Container
  - apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: ${APP_NAME}
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: ${APP_NAME}
      template:
        metadata:
          labels:
            app: ${APP_NAME}
        spec:
          containers:
            - name: ${APP_NAME}
              image: nginx
              ports:
                - containerPort: 80
```



### Conclusion
Modifying OpenShift project templates is a powerful way to standardize application deployments and ensure consistency across projects. By using parameters, object definitions, resource quotas, and limit ranges effectively, you can create flexible and reusable templates tailored to your organization’s needs.



### Configuring a Node Selector for a Project
```yaml
oc adm new-project demo --node-selector "tier=1"
oc annotate namespace demo  openshift.io/node-selector="tier=2" --overwrite
```
### Applying Quotas to Multiple Projects

- The following is an example of creating a cluster resource quota for all projects owned by the qa
user:

```yaml
oc create clusterquota user-qa \
> --project-annotation-selector openshift.io/requester=qa \
> --hard pods=12,secrets=20
```


### The following is an example of creating a cluster resource quota for all projects that have been assigned the environment=qa label:
```yaml
oc create clusterquota env-qa \
> --project-label-selector environment=qa \
> --hard pods=10,services=5
```


## Exposing the Internal OpenShift Registry

To expose the internal OpenShift registry externally:
 ### Verify Registry Status
```bash

oc get pods -n openshift-image-registry
```
### Expose the Registry Service
```bash
oc expose service image-registry -n openshift-image-registry --name=external-registry-route
```
### Retrieve the Route URL
```yaml
oc get route external-registry-route -n openshift-image-registry

# Output should be a URL like: https://external-registry-route.<cluster-domain>
```
### Log in to the Registry using oc CLI
```yaml
oc login -u <username> -p <password>
oc registry login
```
### Alternatively, log in using Docker CLI
```yaml
oc whoami -t  # Retrieve token
docker login -u <username> -p <token> https://external-registry-route.<cluster-domain>
```
### Push and Pull Images
### Push
```yaml
docker tag <image> external-registry-route.<cluster-domain>/<project>/<image>
docker push external-registry-route.<cluster-domain>/<project>/<image>
```
### Pull
```bash
docker pull external-registry-route.<cluster-domain>/<project>/<image>
```


## Overview of the `openshift-config` Namespace in OpenShift

The `openshift-config` namespace in OpenShift is a critical system namespace that stores configuration resources essential for the cluster’s operation, including settings for authentication, networking, and security.

### Key Resources in `openshift-config`

### 1. **Authentication and OAuth Configuration**
   - **OAuth (`OAuth` CR)**: Manages user authentication for the cluster.
     - **Identity Providers**: Configuration for providers like LDAP, GitHub, Google, OpenID Connect, etc.
     - **OAuth Tokens**: Settings for tokens used by applications or users to authenticate with the API.

### 2. **Ingress and Proxy Configuration**
   - **Cluster Proxy (`Proxy` CR)**: Cluster-wide HTTP/HTTPS proxy settings for clusters behind firewalls or with limited internet access.
   - **Cluster Ingress (`Ingress` CR)**: Default settings for external application exposure and the default domain for routes.

### 3. **Network Configuration**
   - **DNS ConfigMap**: Custom DNS configuration required by cluster applications.
   - **Cluster-wide Network Settings**: Configures network policies, network isolation, and other networking settings.

### 4. **Certificate and Secret Management**
   - **Certificates for Internal Services**: TLS certificates for internal cluster services like the API server and internal registry.
   - **Trusted CA Bundle**: CA bundle configuration for trusting external/internal certificates.

### 5. **Image Registry Configuration**
   - **Registry ConfigMap**: Settings for the internal OpenShift image registry (e.g., storage backend and access control).
   - **Image Content Source Policies**: Policies controlling allowed registries for image pulling and storage.

### 6. **Etcd Configuration**
   - **Etcd Settings**: Custom settings for the etcd data store, critical for OpenShift’s cluster state and configuration.

### 7. **Cluster Operators Configuration**
   - Configuration for various cluster operators affecting core components like networking, ingress, and monitoring.

### 8. **Custom Configurations for OpenShift Components**
   - Holds configurations for specific deployment needs, security hardening, or custom operational policies.

### Summary of Key Resource Types in `openshift-config`
   - **ConfigMaps**
   - **Secrets**
   - **Custom Resources**: Like `OAuth`, `Ingress`, and `Proxy`

These configurations in `openshift-config` are essential for managing the functionality, security, and accessibility of the OpenShift cluster.
