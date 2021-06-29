# Advanced Kubernetes For Beginners

<img src="https://user-images.githubusercontent.com/1936716/122274216-ef9f8000-cea7-11eb-8efc-82c7e58cd239.png" width=“700” />

In this workshop we will explore kubernetes beyond simply spinning up a container in a pod.  

## Before starting
Workshop attendees will receave an email with the instance info prior to the workshop.

Notice that training cloud instances will be available only during the workshop and will be terminated **24 hours later**. If you are in our workshop we recommend using the provided cloud instance, you can relax as we have you covered: prerequisites are installed already.

**⚡ IMPORTANT NOTE:**
Everywhere in this repo you see `<YOURADDRESS>` replace with the URL for the instance you were given.  

## Table of content and resources

* [Workshop On YouTube](https://youtu.be/1pkj5tkVozU)
* [Presentation](PDF OF SLIDES HERE)
* [Discord chat](https://discord.gg/TPgUrKUY5Y)

| Title  | Description
|---|---|
| **1 - Getting Connected** | [Instructions](#1-Getting-Connected)  |
| **2 - Control Loop and Operators** | [Instructions](#2-Control-Loop-and-Operators)  |
| **3 - Custom Resource Definitions (CRD)** | [Instructions](#3-Custom-Resource-Definitions-crd)  |
| **4 - Sidecars** | [Instructions](#4-Sidecars)  |
| **5 - ConfigMap and Secrets** | [Instructions](#5-ConfigMap-and-Secrets)  |
| **6 - Resources** | [Instructions](#Resources)  |

## 1. Getting Connected
**✅ Step 1a: The first step in the section.**

In your browser window, navigate to the url <YOURADDRESS>:3000 where your address is the one provided by Alex.
  
When you arrive at the webpage you should be greeted by something similar to this.
<img src="https://user-images.githubusercontent.com/1936716/107884421-a23fe180-6eba-11eb-96d2-4c703ccb1dcf.png" width=“700” />

Click in the `Terminal` menu from the top of the page and select new terminal as shown below
<img src="https://user-images.githubusercontent.com/1936716/107884506-09f62c80-6ebb-11eb-9f7b-42bdb3444cc1.png" width=“700” />

Once you have opened the terminal run
```bash
kubectl get nodes
```

*📃output*

```bash
NAME                          STATUS   ROLES                  AGE   VERSION
learning-cluster-0-master     Ready    control-plane,master   32m   v1.21.1+k3s1
learning-cluster-0-worker-0   Ready    <none>                 31m   v1.21.1+k3s1
learning-cluster-0-worker-1   Ready    <none>                 31m   v1.21.1+k3s1
```
If you see the above output you are ready for the lab.

## 2. Control Loop and Operators
  
To understand operators a bit more we will use kubectl to apply a pre built operator for the Cassandra database.
 
```bash
kubectl apply -f https://raw.githubusercontent.com/datastax/cass-operator/v1.6.0/docs/user/cass-operator-manifests-v1.19.yaml
```

*📃output*
```bash
namespace/cass-operator created
serviceaccount/cass-operator created
secret/cass-operator-webhook-config created
customresourcedefinition.apiextensions.k8s.io/cassandradatacenters.cassandra.datastax.com created
clusterrole.rbac.authorization.k8s.io/cass-operator-cr created
clusterrole.rbac.authorization.k8s.io/cass-operator-webhook created
clusterrolebinding.rbac.authorization.k8s.io/cass-operator-crb created
clusterrolebinding.rbac.authorization.k8s.io/cass-operator-webhook created
role.rbac.authorization.k8s.io/cass-operator created
rolebinding.rbac.authorization.k8s.io/cass-operator created
service/cassandradatacenter-webhook-service created
deployment.apps/cass-operator created
validatingwebhookconfiguration.admissionregistration.k8s.io/cassandradatacenter-webhook-registration created   
```
  
Screenshot of the above working
<img src="https://user-images.githubusercontent.com/blah/blahblah.png" width=“700” />

## 3. Custom Resource Definitions (CRD)
  
Custom resource definitions are ways that you can create assets the function outside of the default kubernetes resource types.  This section pulls heavily from the kubernetes custom resource definition examples in the docs.
 
**✅ Step 3a: Setup CRD.**
  
Before getting started check what CRDs are already on the cluster.
  
```bash
kubectl get crd
```
  
*📃output*
```bash
NAME                              CREATED AT
addons.k3s.cattle.io              2021-06-17T18:59:23Z
helmcharts.helm.cattle.io         2021-06-17T18:59:23Z
helmchartconfigs.helm.cattle.io   2021-06-17T18:59:23Z
```

Next apply the yaml for the new custom CRD.
  
```bash
kubectl apply -f crd.yaml
``` 
  
*📃output*
```bash
customresourcedefinition.apiextensions.k8s.io/crontabs.stable.example.com created
```
  
Check the list of CRDs again.  Notice the new entry for our new asset.
```bash
kubectl get crd
``` 
  
*📃output*
```bash
addons.k3s.cattle.io              2021-06-21T15:43:24Z
helmcharts.helm.cattle.io         2021-06-21T15:43:24Z
helmchartconfigs.helm.cattle.io   2021-06-21T15:43:24Z
crontabs.stable.example.com       2021-06-23T16:10:57Z
```
  
**✅ Step 3b: Setup Pod with the new CRD type.**
  
Start a pod based of the new resource type we defined. 
```bash
kubectl apply -f example-pod.yaml
``` 
  
*📃output*
```bash
crontab.stable.example.com/my-new-cron-object created
```

Check for running resources of the type crontab.
```bash
kubectl get crontab
```   
  
*📃output*
```bash
NAME                 AGE
my-new-cron-object   16s
```

If more information is required running the -o wide flag on the command will provide a full output in yaml format.
```bash
kubectl get ct -o yaml
```  
  
*📃output*
```bash
apiVersion: v1
items:
- apiVersion: stable.example.com/v1
  kind: CronTab
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"stable.example.com/v1","kind":"CronTab","metadata":{"annotations":{},"name":"my-new-cron-object","namespace":"default"},"spec":{"cronSpec":"* * * * */5","image":"my-awesome-cron-image"}}
    creationTimestamp: "2021-06-23T16:11:40Z"
    generation: 1
    name: my-new-cron-object
    namespace: default
    resourceVersion: "157929"
    uid: 43f90a23-13fa-4f60-a673-dc9eff47dc91
  spec:
    cronSpec: '* * * * */5'
    image: my-awesome-cron-image
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

As illistrated here a custom resource can easily be created allowing for much more expanded pod creation.
  
  
## 4. Sidecars
  
Much of the credit for this sidecar section goes to [this great tutorial on Medium](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d)
  
Before we apply anything lets pull the yaml we will use. 
  ```bash
  curl -0 https://gist.githubusercontent.com/bbachi/7d6c40fc8f660eed243f7e9cd31d99c8/raw/03801fa846760d7833ad10f0be69552eede7d828/manifest.yml > sidecars.yaml
```

*📃output*
```bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1478  100  1478    0     0  14211      0 --:--:-- --:--:-- --:--:-- 14211    
```
 
If we open up the yaml file in the file explorer we can see that we are setting up three containers.  One is a simple Nginx server and two are seperate sidecars.  Go ahead and apply the yaml.
 
Note we are creating a deployment instead of a single pod.
  ```bash
kubectl create -f sidecars.yaml
```
  
*📃output*
```bash
deployment.apps/nginx-webapp created
service/nginx-webapp created
```

Kubernetes will create a deployment object that has every component we just set up inside of it.  This deployment object can be created and destroyed by kubectl in a single step.
  
```bash
kubectl get deploy -o wide
```
  
*📃output*
```bash
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS                                             IMAGES                  SELECTOR
nginx-webapp   5/5     5            5           36s   sidecar-container1,sidecar-container2,main-container   busybox,busybox,nginx   app=nginx-webapp
```
  
To see the indavidual pods and ports that are now being used run the following.
  
```bash
kubectl get pods -o wide
```
   
*📃output*
```bash 
 NAME                           READY   STATUS    RESTARTS   AGE    IP           NODE                          NOMINATED NODE   READINESS GATES
nginx-webapp-f5cf77c9c-8789k   3/3     Running   0          101s   10.244.2.6   learning-cluster-0-worker-1   <none>           <none>
nginx-webapp-f5cf77c9c-rm64k   3/3     Running   0          101s   10.244.0.7   learning-cluster-0-master     <none>           <none>
nginx-webapp-f5cf77c9c-9k9vh   3/3     Running   0          101s   10.244.1.7   learning-cluster-0-worker-0   <none>           <none>
nginx-webapp-f5cf77c9c-d76q8   3/3     Running   0          101s   10.244.2.7   learning-cluster-0-worker-1   <none>           <none>
nginx-webapp-f5cf77c9c-5bfdc   3/3     Running   0          101s   10.244.1.6   learning-cluster-0-worker-0   <none>           <none>
```
  
```bash
kubectl get svc -o wide
```
  
*📃output*
```bash
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE     SELECTOR
kubernetes     ClusterIP   10.43.0.1      <none>        443/TCP        20h     <none>
nginx-webapp   NodePort    10.43.218.89   <none>        80:32711/TCP   2m20s   app=nginx-webapp
```
  
We can even see more info if we run the following command.  

```bash
kubectl cluster-info
```
   
*📃output*
```bash
Kubernetes control plane is running at https://127.0.0.1:6443
CoreDNS is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://127.0.0.1:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
  
Since everything is running on localhost we can simply go into the container and hit the web server. Replace YOURCONTAINERID with the container ID of the running webapp. 
  
```bash 
kubectl exec -it YOURCONTAINERID -c main-container -- /bin/sh
```
  
Curl isn't installed in this container so we will have to add it.  To do so run the following.
  
```bash  
apt-get update && apt-get install -y curl
```
  
   
*📃output*
```bash
Get:1 http://security.debian.org/debian-security buster/updates InRelease [65.4 kB]
Get:2 http://deb.debian.org/debian buster InRelease [121 kB]
Get:3 http://deb.debian.org/debian buster-updates InRelease [51.9 kB]
Get:4 http://security.debian.org/debian-security buster/updates/main amd64 Packages [292 kB]
Get:5 http://deb.debian.org/debian buster/main amd64 Packages [7907 kB]
Get:6 http://deb.debian.org/debian buster-updates/main amd64 Packages [10.9 kB]
Fetched 8449 kB in 2s (4806 kB/s)                     
Reading package lists... Done
Reading package lists... Done
Building dependency tree       
Reading state information... Done
curl is already the newest version (7.64.0-4+deb10u2).
0 upgraded, 0 newly installed, 0 to remove and 2 not upgraded.  
```
  
Now we can hit the localhost
```bash
curl localhost
```
   
*📃output*
```bash
...
echo Fri Jun 18 15:31:00 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:00 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:05 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:05 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:10 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:10 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:15 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:15 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:20 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:20 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:25 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:25 UTC 2021 Hi I am from Sidecar container 2
echo Fri Jun 18 15:31:30 UTC 2021 Hi I am from Sidecar container 1
echo Fri Jun 18 15:31:30 UTC 2021 Hi I am from Sidecar container 2
```
Exit the containers shell
  
```bash
exit
```

## 5. ConfigMap and Secrets

**✅ Step 5a: ConfigMaps.**
  
Take a look at what config maps exist on the cluster.
  
```bash
kubectl get cm
``` 
  
   
*📃output*
```bash
NAME               DATA   AGE
kube-root-ca.crt   1      25h
```
  
Use the provided yaml to create a ConfigMap.
  
```bash
kubectl apply -f configmap.yaml
``` 
   
*📃output*
```bash
configmap/configmap-example created
```
  
Check to make sure the ConfigMap was created
  
```bash
kubectl get cm
``` 
   
*📃output*
```bash
NAME                DATA   AGE
kube-root-ca.crt    1      25h
configmap-example   4      20s
```
  
Next create a pod that consumes the new ConfigMap.
  
```bash
kubectl apply -f configmap_example_pod.yaml
```  
   
*📃output*
```bash
pod/configmap-demo-pod created
```
  
Run describe on the pod and look at the references to the configmap.
  
```bash
kubectl describe pods configmap-demo-pod
``` 
   
*📃output*
```bash
  Name:         configmap-demo-pod
Namespace:    default
...      docker.io/library/alpine@sha256:234cb88d3020898631af0ccbbcca9a66ae7306ecd30c9720690858c1b007d2a0
  ...
    Environment:
      FIRST_ENV:         <set to the key 'first_value' of config map 'configmap-example'>      Optional: false
      ASSET_PROPERTIES:  <set to the key 'app_assets_name' of config map 'configmap-example'>  Optional: false
    Mounts:
      /config from config (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-cmqfv (ro)
...
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  6s    default-scheduler  Successfully assigned default/configmap-demo-pod to learning-cluster-0-worker-0
  Normal  Pulling    5s    kubelet            Pulling image "alpine"
  Normal  Pulled     4s    kubelet            Successfully pulled image "alpine" in 843.915004ms
  Normal  Created    4s    kubelet            Created container example
  Normal  Started    4s    kubelet            Started container example
```

  
## 6. Resources
For further reading and labs go to 
[More workshops](https://github.com/MayaLearning) 
[Data On Kubernetes Community and Meetups](dok.community) 
