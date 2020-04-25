# Setting your cluster up

## Prerequisites

```
docker -v
kubectl version
docker-compose -v
```

We suppose here that you have docker and kubernetes installed

## A - KIND

- **A1 - Installation**

Installation procedure for different OS are detailed here
https://kind.sigs.k8s.io/docs/user/quick-start/

```bash

# Installation on MAC
brew install kind

# Check existing Kind clusters
kind get clusters
```

*Expected output*

```
No kind clusters found.
```

- **A2 - Create a KIND cluster**

The Cassandra operator will create a node per worker. As such we need to initialize a cluster with at leat 3 workers. In the following configuration we went up to 5.

```bash
# Apply the configuration (1 master, 5 workers)
kind create cluster --name kind-cassandra --config ./0-setup-your-cluster/01-kind-config.yaml
```

*Expected output*

```
Creating cluster "kind-cassandra" ...
 ✓ Ensuring node image (kindest/node:v1.17.0) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind-cassandra"
You can now use your cluster with:
kubectl cluster-info --context kind-kind-cassandra
Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```

- **A3 Check your k8s cluster**

```bash
# Show the new created cluster
kind get clusters
```

*Expected output*

```
kind-cassandra
```

- **A4 - Link with Kubectl**

```bash
kubectl cluster-info --context kind-kind-cassandra
```

*Expected output*

```
Kubernetes master is running at https://127.0.0.1:45451
KubeDNS is running at https://127.0.0.1:45451/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

- **A5 - List cluster nodes nodes with `kubectl`**

```bash
kubectl get nodes
```

*Expected output*

```
NAME                           STATUS   ROLES    AGE    VERSION
kind-cassandra-control-plane   Ready    master   2m4s   v1.17.0
kind-cassandra-worker          Ready    <none>   86s    v1.17.0
kind-cassandra-worker2         Ready    <none>   88s    v1.17.0
kind-cassandra-worker3         Ready    <none>   88s    v1.17.0
kind-cassandra-worker4         Ready    <none>   88s    v1.17.0
kind-cassandra-worker5         Ready    <none>   88s    v1.17.0
```

## B - NAMESPACE AND STORAGECLASS

- **B1 - Create namespace**

Cass-operator is built to watch over pods running Casandra or DSE in a Kubernetes namespace. Create a namespace for the cluster. For the rest of this guide, we will be using the namespace `cass-operator`. Adjust further commands as necessary to match the namespace you defined.

```
kubectl create ns cass-operator
```

*Expected output*

```
namespace/cass-operator created
```

- **B2 - List storage class**

Kubernetes uses the `StorageClass` resource as an abstraction layer between pods needing persistent storage and the storage resources that a specific Kubernetes cluster can provide. We recommend using the fastest type of networked storage available. Let's create one for your environment.

```
kubectl get storageclass
```

*Expected output*

```
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  11m
```

- **B3 - Describe a storage class**

```
kubectl describe storageclass standard
```

*Expected Ouput*

```
Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

- **B4 - Clone the default storage class and name it `server-storage`**


Sample for my local instance (*to be adapted*).

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
  name: server-storage
provisioner: rancher.io/local-path
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

The above example can be customized to suit your environment and saved as `02-storageclass-kind.yaml`. For the rest of this guide, we'll assume you've defined a `StorageClass` and named it `server-storage`. 

- **B5 - Apply the new policy**

You can apply that file and get the resulting storage classes from Kubernetes.

```bash
kubectl -n cass-operator apply -f ./0-setup-your-cluster/02-storageclass-kind.yaml
```

- **B6 - View policy result**

```
kubectl -n cass-operator get storageClass
```

*Expected Ouput*
```
NAME                       PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
workshop-cassandra-storage (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  31m
standard (default)         rancher.io/local-path   Delete          WaitForFirstConsumer   false                  88m
```

## Congratulations your are set.


Clean up later

```bash
kind delete cluster --name kind-cassandra
```




