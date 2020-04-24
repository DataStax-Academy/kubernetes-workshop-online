# PROMETHEUS

We suppose here that you have docker and kubernetes installed

## A - Installation

```
export namespace=cass-operator
```

```
echo echo $namespace
```

```
./3-prometheus_grafana/30-install-olm.sh 0.14.1
```

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

```bash
kubectl -n cass-operator apply -f ./3-prometheus_grafana/31-install-prometheus.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/32-install-grafana.yaml
```

```bash
kubectl -n cass-operator apply -f ./3-prometheus_grafana/33-prometheus-instance.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/34-prometheus-node-exporter.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/35-prometheus-exporter-service-monitor.yaml
```

```bash
kubectl -n cass-operator apply -f ./3-prometheus_grafana/36-grafana-instance.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/37-grafana-datasource.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/38-grafana-dashboard.yaml
```

```bash
kubectl -n cass-operator port-forward service/grafana-service 3000:3000
kubectl -n cass-operator port-forward service/prometheus-operated 9090:9090
```


## Congratulations your are set.




