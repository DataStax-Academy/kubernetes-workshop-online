kind create cluster --name kind-cassandra --config ./0-setup-your-cluster/01-kind-config.yaml
kubectl cluster-info --context kind-kind-cassandra
kubectl create ns cass-operator
kubectl -n cass-operator apply -f ./0-setup-your-cluster/02-storageclass-kind.yaml
export namespace=cass-operator
./3-prometheus_grafana/install-olm.sh 0.14.1
kubectl -n cass-operator apply -f ./1-cassandra/11-install-cass-operator-v1.1.yaml
kubectl -n cass-operator create -f ./3-prometheus_grafana/prometheus/operator.yaml
kubectl -n cass-operator create -f ./3-prometheus_grafana/grafana/operator.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/prometheus/service_monitor.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/prometheus/instance.yaml
kubectl -n cass-operator createn -f ./3-prometheus_grafana/grafana/datasource.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/dse-analytics.dashboard.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/dse-cluster-condensed.dashboard.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/dse-cluster-metrics.dashboard.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/dse-search-cluster.dashboard.yaml 
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/prometheus-metrics.dashboard.yaml 
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/system-metrics.dashboard.yaml
kubectl -n cass-operator apply -f ./3-prometheus_grafana/grafana/instance.yaml
kubectl -n cass-operator port-forward service/grafana-service 3000:3000&
kubectl -n cass-operator port-forward service/prometheus-operated 9090:9090&
kubectl -n cass-operator apply -f ./3-prometheus_grafana/dse-mini.yaml

