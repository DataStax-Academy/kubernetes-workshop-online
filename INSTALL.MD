# Install Console

```
kubectl apply -f ./2-dashboard/21-install-dashboard.yaml
```
s

kubectl apply -f ./2-dashboard/22-create-dashboard-account.yaml
kubectl apply -f ./2-dashboard/23-create-dashboard-role.yaml
kubectl proxy&

Open a browser on 127.0.0.1:8001
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
```

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')

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



NAME                             READY   STATUS    RESTARTS   AGE
cass-operator-657cb5c695-xqrtf   1/1     Running   0          16m
cluster1-dc1-rack1-sts-0         2/2     Running   0          11m
cluster1-dc1-rack2-sts-0         2/2     Running   0          11m
cluster1-dc1-rack3-sts-0         2/2     Running   0          11m



root@cluster1-dc1-rack1-sts-0:/# nodetool status
Datacenter: dc1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.244.4.4  89.65 KiB  8            68.4%             699442b6-4bcf-4aa3-837e-e269f7af6a9e  rack2
UN  10.244.5.4  70.37 KiB  8            59.0%             defdcd08-9d05-4fe5-a0b0-86a6849fac9f  rack3
UN  10.244.1.3  65.48 KiB  8            72.6%             2beff48d-3628-4a05-8e69-0fe5aa946ea4  rack1





