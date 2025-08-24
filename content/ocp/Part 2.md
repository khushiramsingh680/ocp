---
title: Part 2
weight: 3
date: 2017-01-05
description: Day02
---


## Openshift Authentication and Authorization

- Htpasswd Configuration:
```t
yum install httpd*
htpasswd -c -B -b /tmp/htpasswd student redhat123
htpasswd  -B -b /tmp/htpasswd student2  redhat123

oc create secret generic htpasswd-secret --from-file htpasswd=/tmp/htpasswd -n openshift-config

vi oauth.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
 name: cluster
spec:
 identityProviders:
 - name: ldap
 mappingMethod: claim
 type: HTPasswd
 htpasswd:
 fileData:
 name: htpasswd-secret

```

-  Conplete this as well

```t
https://gitlab.com/gcpnirpendra/openshift-280-training/-/issues/7
```



#### Project

-   How to  create a project 
```
oc new-project myapp
```

-  Check all the project you have got access to 
```
oc get projects
```
-  Check your current project
```
oc project
```

-  Give access to Project  from UI 
-  How to create a pod
```
oc run <name> --image=nginx
```
-  Another example
```
vi pod.yanl
apiVersion: v1
kind: Pod
metadata:
  name: firstpod
spec:
 containers:
  - name: firstcontainer
    image: nginx
```
-  How to check all pods
```
oc get pods
```
-  How to check all pods in all namespaces
```
oc get pods -A
```
-  How to check the name of all pods
```
oc get pods -o name
```
-  How to check the labels of all pods running

```
oc get pods --show-lables
```
-  How to login to a pods

```
oc exec -it podname sh 
```

-  How to check the logs of a pods
```
oc logs podname
```
-  How to login to  a pod
```
oc exec -it podname sh
```
-  How to delete a pod
```
oc delete pod <podname>
```
-  How to delete a pod forcefully
```
oc delete pod --force --grace-period=0
```
-  How to delete multiple pod in one go
```
for i in $(oc get pods -o name ); do oc delete pod $i ;done 
```
-  How to check the logs for a specific container
```
oc logs podname -c <container-name>
```
-  How to login to specific container 
```
oc exec -it podname -c <container-name>
```





#### Service ####

-  How to check all services
```
oc get svc
```
-  How to create a service
```
oc expose pod/deployment  deployment/myapp --port=80
```
-  Service with yaml manifest
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
-  How to check the labels where the service is forwarding requests
```
oc describe svc <svcname>
```
-  How to check the endpoints of a service 
```
oc get ep
```

##### Deployments ####

-  How to create a deployment 
```
oc create deployment myapp --image=nginx
```
-   # Rolling update "www" containers of "frontend" deployment, updating the image
```
oc set image deployment/frontend www=image:v2 
```  
-   # Check the history of deployments including the revision  
```         
oc rollout history deployment/frontend 
```  
-   # Rollback to the previous deployment
```                  
oc rollout undo deployment/frontend  
```
-  # Rollback to a specific revision
```                      
oc rollout undo deployment/frontend --to-revision=2   
```  
-   # Watch rolling update status of "frontend" deployment until completion
```    
oc rollout status -w deployment/frontend 
```     
-   # Rolling restart of the "frontend" deployment 
```             
oc rollout restart deployment/frontend                     
```

#### Secrets ####

-  Check all secrets
```
oc get secrets
```
-  How to create secret 
```
oc create secret tls my-tls-secret \
  --cert=path/to/cert/file \
  --key=path/to/key/file
```
-  Create Secret for username and password
```
oc create secret generic creds --from-literal=name=nippy --from=literal=pass=123
```
-   List the environment variables defined on all pods
```
oc set env pods --all --list
```





  -  # Import environment from a secret
```
  oc set env --from=secret/mysecret dc/myapp
```
 -  # Import environment from a config map with a prefix
```
  oc set env --from=configmap/myconfigmap --prefix=MYSQL_ dc/myapp
```
 -  # Remove the environment variable ENV from container 'c1' in all deployment configs
```
  oc set env deployments --all --containers="c1" ENV-
```


#### ConfigMaps ####

-  List all configmaps
```
oc get cm
```
-  Create a new configmap
```
oc create configmap game-config --from-file=configure-pod-container
```
 -  Get the yaml file for the configmap
```
oc get configmaps game-config -o yaml
```
 -  Use configmap as a volume in pod
```
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never
```


#### Volumes ####

-  emptyDir Example 

```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir:
      sizeLimit: 500Mi
```


-  Check all pvcs
```
oc get pvc
```
-  Create a pvc
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001 
spec:
  capacity:
    storage: 5Gi 
  accessModes:
    - ReadWriteOnce 
```
-   use pvc inside a pod
```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: dockerfile/nginx
      volumeMounts:
      - mountPath: "/var/www/html" 
        name: mypd 
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim 
```






#### Kubernetes Trobleshooting ####
-  Check the nodes status
```
oc get nodes
```
-  Check the status of pods
```
oc get pods 
```
-  Check the pods where are they scheduled
```
oc get pod -o wide
```
-  check the events for a particular namepsace
```
oc get events
```
-   Look for any error in pod
```
oc logs podname
```
-  Describe pod if the pod status is pending
```
oc describe pod podname
```
-   How to login to private image registry
```
docker login
```
-  Create a secret for imagePull secret
```
oc create secret generic regcred \
    --from-file=.dockerconfigjson=<path/to/.docker/config.json> \
    --type=kubernetes.io/dockerconfigjson
```
-  Use Imagepull secret in pod
```
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred

```
 -  Check the ip address of pod with specifice label
```
oc get pods -l app=hostnames \
    -o go-template='{{range .items}}{{.status.podIP}}{{"\n"}}{{end}}'
```
-  Try to check the connectivity from a pod
```
for ep in 10.244.0.5:9376 10.244.0.6:9376 10.244.0.7:9376; do
    wget -qO- $ep
done
```