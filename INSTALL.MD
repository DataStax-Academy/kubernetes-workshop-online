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

### Install DataStax Operator


**Create the namespace**
```
kubectl create ns cass-operator
```

Expected output:
```
namespace/cass-operator created
```

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

## Deploy the Operator

```
kubectl -n cass-operator apply -f ./1/cassandra/11-install-cass-operator-v1.1.yaml
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
kubectl -n workshop-cassandra apply -f ./1-cassandra/12-cassandra-cluster-1nodes.yaml
```




## Scale up your cluster

```
diff ./1-cassandra/12-cassandra-cluster-1nodes.yaml ./1-cassandra/12-cassandra-cluster-3nodes.yaml
```


```
kubectl -n workshop-cassandra get pods
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
kubectl -n workshop-cassandra describe cassdc dc1
```

Sample Output:
```
Name:         dc1
Namespace:    workshop-cassandra
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"cassandra.datastax.com/v1beta1","kind":"CassandraDatacenter","metadata":{"annotations":{},"name":"dc1","namespace":"worksho...
API Version:  cassandra.datastax.com/v1beta1
Kind:         CassandraDatacenter
Metadata:
  Creation Timestamp:  2020-04-23T10:44:25Z
  Finalizers:
    finalizer.cassandra.datastax.com
  Generation:        2
  Resource Version:  5983
  Self Link:         /apis/cassandra.datastax.com/v1beta1/namespaces/workshop-cassandra/cassandradatacenters/dc1
  UID:               6682daeb-ae90-47f2-a67a-7373a2853635
Spec:
  Cluster Name:  cluster1
  Config:
    Cassandra - Yaml:
      Authenticator:  org.apache.cassandra.auth.PasswordAuthenticator
      Authorizer:     org.apache.cassandra.auth.CassandraAuthorizer
      role_manager:   org.apache.cassandra.auth.CassandraRoleManager
    Jvm - Options:
      initial_heap_size:  800M
      max_heap_size:      800M
  Management API Auth:
    Insecure:
  Resources:
  Server Type:     cassandra
  Server Version:  3.11.6
  Size:            1
  Storage Config:
    Cassandra Data Volume Claim Spec:
      Access Modes:
        ReadWriteOnce
      Resources:
        Requests:
          Storage:         5Gi
      Storage Class Name:  cassandra-workshop-storage
Status:
  Cassandra Operator Progress:  Updating
  Last Rolling Restart:         2020-04-23T10:44:25Z
  Node Statuses:
Events:
  Type    Reason           Age                    From           Message
  ----    ------           ----                   ----           -------
  Normal  CreatedResource  2m32s                  cass-operator  Created service cluster1-dc1-service
  Normal  CreatedResource  2m32s                  cass-operator  Created service cluster1-seed-service
  Normal  CreatedResource  2m32s                  cass-operator  Created service cluster1-dc1-all-pods-service
  Normal  CreatedResource  2m32s                  cass-operator  Created statefulset cluster1-dc1-default-sts
  Normal  ScalingUpRack    2m32s (x2 over 2m32s)  cass-operator  Scaling up rack default
```

# Install Console

```
kubectl apply -f ./2-dashboard/21-install-dashboard.yaml
```


kubectl apply -f ./2-dashboard/22-create-dashboard-account.yaml
kubectl apply -f ./2-dashboard/23-create-dashboard-role.yaml
kubectl proxy&

Open a browser on 127.0.0.1:8001
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
```

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')






- Discover your K8s cluster (kind preinstall on machines)

- Create a namespace
- Create the storage class
- Install the operator

- Create a single node cluster 
- node stool status 
- cqlsh
- Scale up to  3 nodes cluster OSS
- change the configuration


- Install kubernetes UI
- Browse the UI
- kill a POD see how it recovers
- Install Promatheuse grafana operator
- Install Prometheus instance, node exporter
- Install Grafana instancem datasource, dashboard
- Show grafana UI
- Install metrics exporter

- Eventually run a DSBENCH
- Show grafana UI



Name:         cass-operator-657cb5c695-xqrtf
Namespace:    cass-operator
Priority:     0
Node:         kind-cassandra-worker5/172.17.0.4
Start Time:   Thu, 23 Apr 2020 16:22:35 +0200
Labels:       name=cass-operator
              pod-template-hash=657cb5c695
Annotations:  <none>
Status:       Running
IP:           10.244.3.2
IPs:
  IP:           10.244.3.2
Controlled By:  ReplicaSet/cass-operator-657cb5c695
Containers:
  cass-operator:
    Container ID:   containerd://64df640066189cf67460330d3a4b8b89f22a7a2196dc849bdcf657bd6a0bd82f
    Image:          datastax/cass-operator:1.1.0
    Image ID:       docker.io/datastax/cass-operator@sha256:26d64588b4e626dcd74c2a6412dff1cfad42c80ad674f67c57f098d5cd20e2dd
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 23 Apr 2020 16:22:41 +0200
    Ready:          True
    Restart Count:  0
    Liveness:       exec [pgrep .*operator] delay=5s timeout=5s period=5s #success=1 #failure=3
    Readiness:      exec [stat /tmp/operator-sdk-ready] delay=5s timeout=5s period=5s #success=1 #failure=1
    Environment:
      WATCH_NAMESPACE:          cass-operator (v1:metadata.namespace)
      POD_NAME:                 cass-operator-657cb5c695-xqrtf (v1:metadata.name)
      OPERATOR_NAME:            cass-operator
      SKIP_VALIDATING_WEBHOOK:  FALSE
    Mounts:
      /tmp/k8s-webhook-server/serving-certs from cass-operator-certs-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from cass-operator-token-jrxc5 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  cass-operator-certs-volume:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  cass-operator-webhook-config
    Optional:    false
  cass-operator-token-jrxc5:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  cass-operator-token-jrxc5
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                             Message
  ----    ------     ----   ----                             -------
  Normal  Scheduled  2m30s  default-scheduler                Successfully assigned cass-operator/cass-operator-657cb5c695-xqrtf to kind-cassandra-worker5
  Normal  Pulling    2m30s  kubelet, kind-cassandra-worker5  Pulling image "datastax/cass-operator:1.1.0"
  Normal  Pulled     2m24s  kubelet, kind-cassandra-worker5  Successfully pulled image "datastax/cass-operator:1.1.0"
  Normal  Created    2m24s  kubelet, kind-cassandra-worker5  Created container cass-operator
  Normal  Started    2m24s  kubelet, kind-cassandra-worker5  Started container cass-operator




kubectl apply -f ./2-dashboard/21-install-dashboard.yaml

kubectl apply -f ./2-dashboard/22-create-dashboard-account.yaml

kubectl apply -f ./2-dashboard/23-create-dashboard-role.yaml

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}’)











