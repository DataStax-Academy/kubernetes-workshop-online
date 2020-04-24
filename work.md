kind create cluster --name kind-cassandra --config ./0-setup-your-cluster/01-kind-config.yaml

kubectl cluster-info --context kind-kind-cassandra

kubectl create ns cass-operator

kubectl -n cass-operator apply -f ./0-setup-your-cluster/02-kind-storageClass.yaml

kubectl -n cass-operator apply -f ./1-cassandra/11-install-cass-operator-v1.1.yaml

kubectl -n cass-operator get pod

kubectl describe pods my-pod cass-operator-657cb5c695-xqrtf

kubectl -n cass-operator apply -f ./1-cassandra/12-cassandra-cluster-nodes-racks.yaml

#Simple
kubectl -n cass-operator apply -f ./1-cassandra/12-cassandra-cluster-1nodes.yaml
diff ./1-cassandra/12-cassandra-cluster-1nodes.yaml ./1-cassandra/12-cassandra-cluster-3nodes.yaml 
kubectl -n cass-operator apply -f ./1-cassandra/12-cassandra-cluster-3nodes.yaml

kubectl -n cass-operator describe cassdc dc1

kubectl -n cass-operator get pod

kubectl -n cass-operator get secret cluster1-superuser -o yaml

echo Y2x1c3RlcjEtc3VwZXJ1c2Vy | base64 -d
echo VjJ5dXJ5bUk4eldUZkZPYzFveDBOQ21iNHdSTl9Bd0dNZWpoNGFrU1hFblgtZTNZVTFlRWpR | base64 -d

kubectl -n cass-operator exec -it cluster1-dc1-rack1-sts-0 -- /bin/bash

nodetool status

cqlsh cluster1-dc1-service.cass-operator -u cluster1-superuser -p V2yurymI8zWTfFOc1ox0NCmb4wRN_AwGMejh4akSXEnX-e3YU1eEjQ

# Install the prometheus operator
kubectl -n cass-operator apply -f ./3-prometheus/31-install-prometheus.yaml

# Provision a prometheus instance (in this case 2 pod deployment)
kubectl -n cass-operator apply -f ./3-prometheus/32-install-node-exporter.yaml


kubectl -n cass-operator apply -f ./4-grafana/41-install-grafana.yaml

