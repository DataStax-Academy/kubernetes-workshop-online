![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/00-screenplay.png?raw=true)

# Monitor a Cluster

At this point we expect you to have successfully complete section #0 and section #1. Section #2 is all about visualization and optional.

## 1. Clean up cluster defined in previous sections

In this section we want to illustrate the `Dse Metrics Exporter`. As such we would need to run a cluster of DataStax Enterprise nodes and not Cassandra OSS node (yet). We will clean up the `dc1` we created in previous section. In the CRD there is a property named `stopped` to set to true and apply the new yaml, that's it.

📘 **1a. Check what's new in the yaml file**
```bash
diff ./1-cassandra/14-cassandra-cluster-3nodes-newconfig.yaml ./1-cassandra/15-cassandra-cluster-stop.yaml 
```
📗 **Expected output**
```
>   stopped: true
22c23
<   config:    
---
>   config:
```
📘 **1b. Properly stop the Cassandra `dc1` datacenter**
```bash
kubectl -n cass-operator apply -f ./1-cassandra/15-cassandra-cluster-stop.yaml
```
📘 **1c. Monitor progression in the command line**
```bash
watch kubectl -n cass-operator get pods
```
📘 **1d. Delete the Cassandra `dc1` datacenter**
```bash
kubectl -n cass-operator delete -f ./1-cassandra/15-cassandra-cluster-stop.yaml
```
📘 **1e. Monitor progression in the command line**
```bash
watch kubectl -n cass-operator get pods
```

```
kubectl get svc -n cass-operator --show-labels=true
```
## 2. Install Operator Lifecycle Manager (OLM)

This project is a component of the Operator Framework, an open source toolkit to manage Kubernetes native applications, called Operators, in an effective, automated, and scalable way. Read more in the introduction blog post and learn about practical use cases at OLM-Book.

OLM extends *Kubernetes* to provide a declarative way to install, manage, and upgrade Operators and their dependencies in a cluster. You can find more information in the [reference documentation](https://github.com/operator-framework/operator-lifecycle-manager)

📘 **2a. Set env var `namespace`**
```bash
export namespace=cass-operator
echo $namespace
```

📘 **2b.Install OLM**
```bash
./3-prometheus_grafana/install-olm.sh 0.14.1
```
📗 **Expected output**
```
customresourcedefinition.apiextensions.k8s.io/clusterserviceversions.operators.coreos.com created
customresourcedefinition.apiextensions.k8s.io/installplans.operators.coreos.com created
customresourcedefinition.apiextensions.k8s.io/subscriptions.operators.coreos.com created
customresourcedefinition.apiextensions.k8s.io/catalogsources.operators.coreos.com created
customresourcedefinition.apiextensions.k8s.io/operatorgroups.operators.coreos.com created
namespace/olm created
namespace/operators created
serviceaccount/olm-operator-serviceaccount created
clusterrole.rbac.authorization.k8s.io/system:controller:operator-lifecycle-manager created
clusterrolebinding.rbac.authorization.k8s.io/olm-operator-binding-olm created
deployment.apps/olm-operator created
deployment.apps/catalog-operator created
clusterrole.rbac.authorization.k8s.io/aggregate-olm-edit created
clusterrole.rbac.authorization.k8s.io/aggregate-olm-view created
operatorgroup.operators.coreos.com/global-operators created
operatorgroup.operators.coreos.com/olm-operators created
clusterserviceversion.operators.coreos.com/packageserver created
catalogsource.operators.coreos.com/operatorhubio-catalog created
Waiting for deployment "olm-operator" rollout to finish: 0 of 1 updated replicas are available...
deployment "olm-operator" successfully rolled out
Waiting for deployment "catalog-operator" rollout to finish: 0 of 1 updated replicas are available...
deployment "catalog-operator" successfully rolled out
Package server phase: InstallReady
Package server phase: Installing
Package server phase: Succeeded
deployment "packageserver" successfully rolled out
```

## 3. Install Prometheus Operator

The Prometheus Operator makes the Prometheus configuration Kubernetes native and manages and operates Prometheus and Alertmanager clusters. It is a piece of the puzzle regarding full end-to-end monitoring. 

`kube-prometheus` combines the Prometheus Operator with a collection of manifests to help getting started with monitoring Kubernetes itself and applications running on top of it. You can find more information in the [reference documentation](https://github.com/coreos/prometheus-operator)

📘 **3a. Install Operator**

```bash
kubectl -n cass-operator create -f ./3-prometheus_grafana/prometheus/operator.yaml
```

📘 **3b. Install the instance of prometheus**
```
kubectl -n cass-operator apply -f ./3-prometheus_grafana/prometheus/instance.yaml
```

📘 **3c. Install Service Monitor**
```
kubectl -n cass-operator apply -f ./3-prometheus_grafana/prometheus/service_monitor.yaml
```


## 4. Install Grafana Operator

A Kubernetes Operator based on the Operator SDK for creating and managing Grafana instances. The Operator is available on Operator Hub. It can deploy and manage a Grafana instance on Kubernetes and OpenShift. The following features are supported:
- Install Grafana to a namespace
- Import Grafana dashboards from the same or other namespaces
- Import Grafana datasources from the same namespace
- Install Plugins (panels) defined as dependencies of dashboards

📘 **4a. Install Operator**
```bash
kubectl -n cass-operator create -f ./3-prometheus_grafana/grafana/operator.yaml
```

📘 **4b.Install DataSource to read from prometheus**
```bash
kubectl -n cass-operator create -f ./3-prometheus_grafana/grafana/datasource.yaml
```

📘 **4c.Install dashboards for DSE and prometheus**
```bash
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/dse-analytics.dashboard.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/dse-cluster-condensed.dashboard.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/dse-cluster-metrics.dashboard.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/dse-search-cluster.dashboard.yaml 
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/prometheus-metrics.dashboard.yaml 
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/system-metrics.dashboard.yaml
```

📘 **4d.Install Grafana instance**
```bash
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/instance.yaml
```

📘 **4e. Show labels in the namespace**
```
kubectl get svc -n cass-operator --show-labels=true
```
📗 **Expected output**
```
NAME                                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE     LABELS
cass-operator-metrics                 ClusterIP   10.96.254.17    <none>        8383/TCP,8686/TCP   10m     name=cass-operator
cassandradatacenter-webhook-service   ClusterIP   10.96.44.146    <none>        443/TCP             12m     name=cass-operator-webhook
grafana-operator-metrics              ClusterIP   10.96.104.188   <none>        8080/TCP            10m     name=grafana-operator
grafana-service                       ClusterIP   10.96.240.129   <none>        3000/TCP            7s      <none>
prometheus-operated                   ClusterIP   None            <none>        9090/TCP            4m56s   operated-prometheus=true
```

📘 **4f. Forward Ports**

Note: In cloud based environment you may need to access dedicated URL and not port 3000 as we forwarded url to cope with security constraints.

```bash
kubectl -n cass-operator port-forward --address 0.0.0.0 service/grafana-service 3000:3000&
```

```bash
kubectl -n cass-operator port-forward --address 0.0.0.0 service/prometheus-operated 9090:9090&
```

## 5. Start DSE Cluster

📘 **5a.Start the 3 DSE nodes clusters**

This could be quite CPU consuming reduce the size of the cluster to `1` if you in the `dse-mini.yaml` if you are working on a machine more than 4 years old.


```bash
kubectl -n cass-operator apply -f ./3-prometheus_grafana/dse/dse-mini.yaml
```

📘 **5b.Watch evolution it might need some time**
```bash
watch kubectl -n cass-operator get pods -l cassandra.datastax.com/cluster=cluster2
```
📗 **Expected output after a few min**
```
NAME                         READY   STATUS    RESTARTS   AGE
cluster2-dc2-default-sts-0   2/2     Running   0          6m55s
cluster2-dc2-default-sts-1   2/2     Running   0          6m55s
cluster2-dc2-default-sts-2   2/2     Running   0          6m55s
```

📘 **5c.Have a look in the logs of a pod**

```bash
kubectl -n cass-operator logs cluster2-dc2-default-sts-0 -c cassandra
```

## 6. Browse DSE Cluster

We will redo the command from section 1 to enter the cluster and show some information we may go faster here.

📘 **6a. Extract credentials**
```bash
kubectl -n cass-operator get secret cluster2-superuser -o yaml
```
📗 **Sample output**
```yaml
apiVersion: v1
data:
  password: SHREUXBsLUlMRXNJeDl1Skdla0FKQ1dCYVBhOHFhWUJwS3gwRzBucHdIVENkLUlnU3VCcFdB
  username: Y2x1c3RlcjItc3VwZXJ1c2Vy
kind: Secret
metadata:
  creationTimestamp: "2020-04-25T12:05:30Z"
  name: cluster2-superuser
  namespace: cass-operator
  resourceVersion: "31756"
  selfLink: /api/v1/namespaces/cass-operator/secrets/cluster2-superuser
  uid: 70b7b4ad-494a-4e27-a566-d7e935cdeabc
type: Opaque
```

📘 **6b. Get user/password**
```
echo Y2x1c3RlcjItc3VwZXJ1c2Vy | base64 -d && echo ""
echo SHREUXBsLUlMRXNJeDl1Skdla0FKQ1dCYVBhOHFhWUJwS3gwRzBucHdIVENkLUlnU3VCcFdB | base64 -d && echo ""
```

📘 **6c. Enter the node and show status**
```bash
kubectl -n cass-operator exec -it cluster2-dc2-default-sts-0  -- /bin/bash
dsetool status
```
📗 **Sample output**
```bash
DC: dc2             Workload: Cassandra       Graph: no     
======================================================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--   Address          Load             Effective-Ownership  Token                                        Rack         Health [0,1] 
                                                            5265284122577758891                                                    
UN   10.244.1.6       105.82 KiB       59.34%               -8221998853083547597                         default      0.40         
UN   10.244.4.9       158.14 KiB       67.55%               -720713414843373246                          default      0.30         
UN   10.244.2.10      136.32 KiB       73.11%               5265284122577758891                          default      0.40         
```
You can note that now we are working with a `dc2` but not part of the same cluster as before.

📘 **6d. CQLSH...**
```bash
cqlsh cluster2-dc2-service -u cluster2-superuser -p HtDQpl-ILEsIx9uJGekAJCWBaPa8qaYBpKx0G0npwHTCd-IgSuBpWA
```
```sql
DESCRIBE KEYSPACES;
```
📘 **6e.Go back to `kubectl`**
```
exit
exit
```

## 7. Let's blow your mind

If you show UI from grafana and prometheus you might notice than the screens are still empty. This is **expected**. Let's enable metrics collections by adding a couple of line to the dse cluster definition `yaml`

```yaml
10-write-prom-conf:
  enabled: true
  port: 9103
  staleness-delta: 300
```

📘 **7a. Apply this configuration**
```bash
kubectl -n cass-operator apply -f ./3-prometheus_grafana/dse/dse-mini-enablemetrics.yaml
```

📘 **7b. Apply this configuration**
Wait for the configuration to be deploy in round robin
```bash
watch kubectl -n cass-operator get pods -l cassandra.datastax.com/cluster=cluster2
```

📘 **7c. Prometheus target**

Prometheus UI should be available at `http://localhost:9090` but it mays vary on the cloud instance depending on port forwarding and coping with security on EC2 machines.

You can navigate to `Status` dropdown in the topbar and select `target` to see the following:

- The cluster is detected but nodes are not sending metrics yet:

![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/prometheus-target-empty.png?raw=true)

- Let the party begin

![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/prometheus-target-up.png?raw=true)

- You can now graph some metrics withing prometheus 

![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/prometheus-graph-up.png?raw=true)

📘 **7d. Grafana UI**

Grafana should be available at `http://localhost:3000` but it mays vary on the cloud instance depending on port forwarding and coping with security on EC2 machines. Login and password are provided in the `yaml` files, they are `admin` and `secret`.

- Metrics starting to pull to Grafana

![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/grafana-starting.png?raw=true)

- After a bit of time

![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/grafana-jvm.png?raw=true)

- You can browse dashboards there are a lot, here is the JVM. Only the dashboards using the datasource we created before will have data. The one starting with `dse`.

![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/grafana-up.png?raw=true)

## 8. Stop and clean up

As before you may want to stop the cluster properly and then stop everything else with the following commands.

## 8a. Stop the DSE cluster properly
```bash
kubectl -n cass-operator apply -f ./3-prometheus_grafana/dse/dse-mini-stop.yaml
```

## 8b. Delete resources associated

```bash
kubectl -n cass-operator delete -f ./3-prometheus_grafana/dse/dse-mini-stop.yaml
```

## 8c. Delete the namespace and clean up the rest of the artifacts
```bash
kubectl delete ns cass-operator
```

## 8d. Delete the kub cluster
```bash
kind delete cluster  --name=kind-cassandra
```

## Congratulations, your are done.

**We love your feedback, please consider to stay with us for the Mentimeter live survey**

![OK](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/welldone.jpg?raw=true)
