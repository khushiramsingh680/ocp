---
title: Use and configure OpenID connect with Azure AD
weight: 3
date: 2017-01-05
description: Use and configure OpenID connect with Azure AD
---


Use and configure OpenID connect with Azure AD


# Use and configure OpenID connect with Azure AD

## Required informations 

| key | env | value |
|---  | --- | ---   |
| service principal | staging    | "RedHat - Openshift /   (auth)" () |

## Azure AD service principal requirements

- Ensure the service principal has the following API permissions
  `Microsoft Graph / "User.Read"`

- Ensure it has been "Granted for, .

- Ensure it has been promoted as Enterprise Application, otherwise raise an internal IT request to get this requirement. It could allow to restrict the access to a group of users that is very useful.

- Ensure the redirect URI is properly set, it must match the following pattern: `https://oauth-openshift.apps.<fullclustername>/oauth2callback/azuread`. Of course substitute `<fullclustername>` by the one you set in the IPI configuration file.
	

## Openshift settings

- Create an openshift secret  
_be sure to substitute the `<application registration secret>` by the right value !_

```bash
oc create secret generic azuread --from-literal=clientSecret=<app registration secret>  -n openshift-config
```

- Create an `oauth-config.yaml` file to configure oauth. 

_be sure to substitute the `<application id>` by the right value !_

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  annotations:
    release.openshift.io/create-only: "true"
  generation: 1
  name: cluster
  selfLink: /apis/config.openshift.io/v1/oauths/cluster
spec:
  identityProviders:
  - mappingMethod: claim
    name: azuread
    openID:
      claims:
        email:
        - email
        id:
        - sub
        name:
        - name
        preferredUsername:
        - email
      clientID: <application id>
      clientSecret:
        name: azuread
      issuer: https://login.microsoftonline.com/8b87af7d-8647-4dc7-8df4-5f69a2011bb5/v2.0
    type: OpenID
```

- apply the config

```bash
oc apply -f oauth-config.yaml
```

## Post configuration tests

- wait a moment and try to login in a new private window of your internet browser.  
`azuread` should be an option available and it must work properly.

## Post configuration steps

- signin as kubeadmin and promote your cloud admin account as `cluster-admin`

Example:
```bash
oc adm policy add-cluster-role-to-user cluster-admin user-id
```

- logout and login with your own admin account. Ensure you have the cluster-admin role and remove the kubeadmin secret

```bash
oc delete secrets kubeadmin -n kube-system
```

- logout and ensure the kube:admin option is not available anymore.

- if you need to restrict access to the cluster, go on the Azure AD, select your entreprise application, go in the "Users and Groups" blade, and select the ones who could access to the instance.



[Please click to configure Azure Authentication from UI](https://docs.microsoft.com/en-us/azure/openshift/configure-azure-ad-ui)
