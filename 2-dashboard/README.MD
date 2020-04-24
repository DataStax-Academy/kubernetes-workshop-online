# Kubernetes UI

We suppose here that you have docker and kubernetes installed

## A - Installation

- **A1 - Installation**

More information on the dashboard can be found here : 
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta8/aio/deploy/recommended.yaml
```

*Expected output*

```
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

- **A2 - SECURITY**

You will need you to create a user and the related role

```bash
kubectl apply -f ./2-dashboard/22-create-dashboard-account.yaml
kubectl apply -f ./2-dashboard/23-create-dashboard-role.yaml
```

- **A3 - START the UI**

```
kubectl proxy
```

Open a browser on 127.0.0.1:8001

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
```

- **A3 - Connect and Browse to the UI**

Using the following command line you will create a token to connect.

```bash
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

## Congratulations your are set.




