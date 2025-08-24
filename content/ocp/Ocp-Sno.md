---
title: OpenShift Single Node Cluster
description: Step-by-step guide for deploying and integrating OpenShift Single Node clusters.
weight: 1
icon: fab fa-openshift
---

## OpenShift Single Node Installation Guide (Agent-Based Assisted Installation)

### Prerequisites
- **Virtualization Software** (choose one):
  - VMware Workstation  
  - VirtualBox  
  - Hyper-V  
  - VMware vSphere  
- **Virtual Machine Requirements**: Minimum **8 vCPUs**, **32 GB RAM**, and **120 GB storage** (SSD recommended).  
- **Developer Account** on Red Hat: [cloud.redhat.com](https://cloud.redhat.com)

---

### Installation Steps

1. Log in to **[cloud.redhat.com](https://cloud.redhat.com)** and navigate to **Services**.  
   ![ocp100](/images/ocp100.png)

2. Select **OpenShift** and click **Create Cluster**.  
   ![ocp101](/images/ocp101.png)  
   ![ocp102](/images/ocp102.png)  
   ![ocp104](/images/ocp104.png)

3. Fill in the required details.  
   ![ocp105](/images/ocp105.png)

4. Provide your **SSH public key** and download the generated **ISO image**.  
   ![ocp106](/images/ocp106.png)

5. Create a new VM using the downloaded ISO image.  
   - Assign **8 vCPUs**, **32 GB RAM**, and **120 GB storage**.  

6. Once the cluster is deployed, the details will be visible in the **Red Hat Console**.  
   ![ocp107](/images/ocp107.png)

7. Download the **kubeconfig file** and **kubeadmin credentials**.  
   ![ocp108](/images/ocp108.png)

---

## Post-Installation Tasks

(You can include cluster configuration, storage setup, monitoring, and user management here.)

---

## OpenShift and Azure AD Integration

### Prerequisites
- **Azure AD Tenant** with required user groups.  
- A running **OpenShift cluster**.  

---

### Integration Steps

#### 1. Register an Application in Azure AD
1. Log in to the **Azure Portal**.  
2. Navigate to **Azure Active Directory** → **App registrations** → **New registration**.  
3. Enter a name (e.g., `OpenShiftApp`).  
4. Set the **Redirect URI** to the OpenShift callback URL:  
   `https://<openshift-cluster>/oauth2callback`  
5. Click **Register**.  

---

#### 2. Configure Azure AD for OpenShift OAuth

**2.1 Create a Client Secret**  
- In the registered app, go to **Certificates & Secrets** → **New client secret**.  
- Add a description, choose an expiration period, and click **Add**.  
- Copy and securely store the secret value.  

**2.2 Assign API Permissions**  
- Navigate to **API permissions** → **Add a permission**.  
- Select **Microsoft Graph** → **Delegated permissions**.  
- Add `User.Read` and any other required permissions.  

---

#### 3. Configure OpenShift OAuth for Azure AD

On the OpenShift master node, edit the OAuth configuration:

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
  annotations:
    release.openshift.io/create-only: "true"
spec:
  identityProviders:
  - name: azuread
    mappingMethod: claim
    type: OpenID
    openID:
      clientID: <application-id>
      clientSecret:
        name: azuread
      issuer: https://login.microsoftonline.com/<tenant-id>/v2.0
      claims:
        email:
        - email
        id:
        - sub
        name:
        - name
        preferredUsername:
        - email
```