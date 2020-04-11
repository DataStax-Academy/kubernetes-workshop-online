## Setting your cluster up


### Install Kind
```
brew install kind
```

### Create a kub cluster (1 master, 3 workers)
```
kind create cluster --name kind-cassandra --config 1-kind-config.yaml
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
kind-cassandra-control-plane   Ready    master   101s   v1.17.0
kind-cassandra-worker          Ready    <none>   67s    v1.17.0
kind-cassandra-worker2         Ready    <none>   70s    v1.17.0
kind-cassandra-worker3         Ready    <none>   74s    v1.17.0
```

### Install DataStax Operator


**Create the namespace**
```
kubectl create ns my-db-ns
```

namespace/my-db-ns created

**List storage class**
```
kubectl get storageclass
```

NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  11m


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


Clone the default storage class and name it server-storage
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
kubectl -n my-db-ns apply -f ./2-kind-storageClass.yaml
```

List new storage classes
```
kubectl -n my-db-ns get storageClass
```

Results:
```
NAME                       PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
server-storage (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  31m
standard (default)         rancher.io/local-path   Delete          WaitForFirstConsumer   false                  88m
```

## Deploy the Operator

```
kubectl -n my-db-ns apply -f ./3-cass-operator-manifests.yaml
```

Output
```
serviceaccount/cass-operator created
role.rbac.authorization.k8s.io/cass-operator created
rolebinding.rbac.authorization.k8s.io/cass-operator created
customresourcedefinition.apiextensions.k8s.io/cassandradatacenters.cassandra.datastax.com created
deployment.apps/cass-operator created
```


## Create the cluster

```
kubectl -n my-db-ns apply -f ./4-cassdc-cassandra.yaml
```

```
kubectl -n my-db-ns get pods
```

Output
```
NAME                            READY   STATUS            RESTARTS   AGE
cass-operator-56f5f8c7c-4gzwz   1/1     Running           0          7m13s
cluster1-dc1-r1-sts-0           0/2     PodInitializing   0          36s
cluster1-dc1-r2-sts-0           0/2     PodInitializing   0          36s
cluster1-dc1-r3-sts-0           0/2     PodInitializing   0          36s
```

Starting....

```
NAME                            READY   STATUS    RESTARTS   AGE
cass-operator-56f5f8c7c-4gzwz   1/1     Running   0          11m
cluster1-dc1-r1-sts-0           1/2     Running   0          5m3s
cluster1-dc1-r2-sts-0           1/2     Running   0          5m3s
cluster1-dc1-r3-sts-0           1/2     Running   0          5m3s
```

Get you 
```
kubectl -n my-db-ns describe cassdc dc1
```


```
Name:         dc1
Namespace:    my-db-ns
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"cassandra.datastax.com/v1alpha2","kind":"CassandraDatacenter","metadata":{"annotations":{},"name":"dc1","namespace":"my-db-...
API Version:  cassandra.datastax.com/v1alpha2
Kind:         CassandraDatacenter
Metadata:
  Creation Timestamp:  2020-03-25T15:48:52Z
  Finalizers:
    finalizer.cassandra.datastax.com
  Generation:        2
  Resource Version:  19358
  Self Link:         /apis/cassandra.datastax.com/v1alpha2/namespaces/my-db-ns/cassandradatacenters/dc1
  UID:               2224bbe4-5dd6-4757-ad5e-07065d247e6d
Spec:
  Cluster Name:  cluster1
  Config:
    Cassandra - Yaml:
      file_cache_size_in_mb:  100
      memtable_space_in_mb:   100
      num_tokens:             8
    Jvm - Options:
      initial_heap_size:  1g
      max_heap_size:      1g
  Config Builder Image:   datastaxlabs/dse-k8s-config-builder:0.9.0-20200316
  Image Version:          3.11.6
  Management API Auth:
    Insecure:
  Racks:
    Name:  r1
    Name:  r2
    Name:  r3
  Resources:
  Server Image:  datastaxlabs/apache-cassandra-with-mgmtapi:3.11.6-20200316
  Server Type:   cassandra
  Size:          3
  Storage Config:
    Cassandra Data Volume Claim Spec:
      Access Modes:
        ReadWriteOnce
      Resources:
        Requests:
          Storage:         5Gi
      Storage Class Name:  server-storage
Status:
  Cassandra Operator Progress:  Updating
  Last Server Node Started:     2020-03-25T15:50:18Z
Events:
  Type     Reason           Age    From                Message
  ----     ------           ----   ----                -------
  Normal   CreatedResource  8m42s  cassandra-operator  Created service cluster1-dc1-service
  Normal   CreatedResource  8m42s  cassandra-operator  Created service cluster1-seed-service
  Normal   CreatedResource  8m42s  cassandra-operator  Created service cluster1-dc1-all-pods-service
  Warning  ReconcileFailed  8m42s  cassandra-operator  services "cluster1-dc1-all-pods-service" already exists
  Normal   CreatedResource  8m41s  cassandra-operator  Created statefulset cluster1-dc1-r1-sts
  Normal   CreatedResource  8m41s  cassandra-operator  Created statefulset cluster1-dc1-r2-sts
  Normal   CreatedResource  8m41s  cassandra-operator  Created statefulset cluster1-dc1-r3-sts
```

# Install Console

```
kubectl apply -f ./5-kubernetes-ui.yaml
```


kubectl -n kubernetes-dashboard apply -f ./6-ui-account.yaml


kubectl -n kubernetes-dashboard apply -f ./7-ui-role.yaml

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

kubectl proxy



kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}’)













