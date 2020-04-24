## Setting your cluster up

### Install kind

```
brew install kind
```

### Check clusters for kinds
```
kind get clustets
```

Expected output:
```
No kind clusters found.
```

### Create a kub cluster (1 master, 3 workers)
```
kind create cluster --name kind-cassandra --config ./0-setup-your-cluster/01-kind-config.yaml
```

Expected output:
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

### Setup docker

# Check your k8s cluster

```
kind get clusters
```

Expected output:
```
kind-cassandra
```

```
kubectl cluster-info --context kind-kind-cassandra
```

Expected output:
```
Kubernetes master is running at https://127.0.0.1:45451
KubeDNS is running at https://127.0.0.1:45451/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```
kubectl get nodes
```

Expected output:
```
NAME                           STATUS   ROLES    AGE    VERSION
kind-cassandra-control-plane   Ready    master   2m4s   v1.17.0
kind-cassandra-worker          Ready    <none>   86s    v1.17.0
kind-cassandra-worker2         Ready    <none>   88s    v1.17.0
kind-cassandra-worker3         Ready    <none>   88s    v1.17.0
kind-cassandra-worker4         Ready    <none>   88s    v1.17.0
kind-cassandra-worker5         Ready    <none>   88s    v1.17.0
```

### Create namespace


**Create the namespace**
```
kubectl create ns cass-operator
```

Expected output:
```
namespace/cass-operator created
```

### List storage class

```
kubectl get storageclass
```

Expected output
```
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  11m
```

Describe the storage class
```
kubectl describe storageclass standard
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


Clone the default storage class and name it `server-storage`
```
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

Apply the new policy
```
kubectl -n cass-operator apply -f ./0-setup-your-cluster/02-kind-storageClass.yaml
```

List new storage classes
```
kubectl -n cass-operator get storageClass
```

Results:
```
NAME                       PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
workshop-cassandra-storage (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  31m
standard (default)         rancher.io/local-path   Delete          WaitForFirstConsumer   false                  88m
```








