# Install Cassandra

We suppose here that you have docker, kubernetes and kind ready

## A - INSTALL OPERATOR

- **A1 - Install the operator**

```bash
# Apply the manisfest
kubectl -n cass-operator apply -f ./1-cassandra/11-install-cass-operator-v1.1.yaml
```

*Expected output*

```
serviceaccount/cass-operator created
role.rbac.authorization.k8s.io/cass-operator created
rolebinding.rbac.authorization.k8s.io/cass-operator created
customresourcedefinition.apiextensions.k8s.io/cassandradatacenters.cassandra.datastax.com created
deployment.apps/cass-operator created
```

- **A2 - Look at created pod**

```bash
kubectl -n cass-operator get pod
```

*Expected output*

```
tbd
```

- **A3 - Describe operator pod, notive the CRD**

```bash
kubectl describe pods my-pod cass-operator-657cb5c695-xqrtf
```

*Expected output*

```
tbd
```

## B - CREATE THE CLUSTER


- **B1 - Create a cluster**

```
kubectl -n cass-operator apply -f ./1-cassandra/12-cassandra-cluster-nodes-racks.yaml
```

*Expected output*

```
tbd
```

- **B2 - Check we are ok**

```bash
kubectl -n cass-operator get pod
```

*Expected output*

```
# During initialization
NAME                            READY   STATUS            RESTARTS   AGE
cass-operator-56f5f8c7c-4gzwz   1/1     Running           0          7m13s
cluster1-dc1-r1-sts-0           0/2     PodInitializing   0          36s
cluster1-dc1-r2-sts-0           0/2     PodInitializing   0          36s
cluster1-dc1-r3-sts-0           0/2     PodInitializing   0          36s

# When ready
NAME                            READY   STATUS    RESTARTS   AGE
cass-operator-56f5f8c7c-4gzwz   1/1     Running   0          11m
cluster1-dc1-r1-sts-0           1/2     Running   0          5m3s
cluster1-dc1-r2-sts-0           1/2     Running   0          5m3s
cluster1-dc1-r3-sts-0           1/2     Running   0          5m3s
```

 - **B3 - Describe the DataCenter**


```bashs
kubectl -n workshop-cassandra describe cassdc dc1
```

*Expecte output*

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


**Cluster and DataCenter:**

A logical `datacenter` is the primary resource managed by the cass-operator. Within a single Kubernetes namespace:

- A single `CassandraDatacenter `resource defines a single-datacenter cluster.
- Two or more CassandraDatacenter resources with different clusterName's define separate and unrelated single-datacenter clusters. Note the operator manages both clusters since they reside within the same Kubernetes namespace.
- Two or more CassandraDatacenter resources that have the same clusterName define a multi-datacenter cluster. The operator will join the instances in each datacenter into a logical topology that acts as a single cluster.

For this guide, we define a single-datacenter cluster. The cluster is named cluster1 with the datacenter named dc1.

**Racks**

Cassandra is rack-aware, and the racks parameter will configure the operator to set up pods in a rack aware way. Note the Kubernetes worker nodes must have labels matching `failure-domain.beta.kubernetes.io/zone`. Racks must have identifiers. In this guide we will use `r1`, `r2`, and `r3`.

**NodeCount**

The size parameter is the number of nodes to run in the datacenter. For optimal performance, it's recommended to run only one server instance per Kubernetes worker node. The operator will enforce that limit, and pods may get stuck in the Pending status if there are insufficient Kubernetes workers available.


## C - Connect to the cluster

- **C1 - First we will need user and password coming form the secret**

By default, a cassandra superuser gets created by the operator. A Kubernetes secret will be created for it, named `<cluserName>-superuser`. It will contain username and password keys.

```bash
kubectl -n cass-operator get secret cluster1-superuser -o yaml
```

*Expected output*

```
cedricklunven@clunhost:~/dev/WORKSPACES/DATASTAX/kubernetes-workshop-online> kubectl -n cass-operator get secret cluster1-superuser -o yaml
apiVersion: v1
data:
  password: VjJ5dXJ5bUk4eldUZkZPYzFveDBOQ21iNHdSTl9Bd0dNZWpoNGFrU1hFblgtZTNZVTFlRWpR
  username: Y2x1c3RlcjEtc3VwZXJ1c2Vy
```

Those values are encoded in base64, we need to decode:

```bash
echo Y2x1c3RlcjEtc3VwZXJ1c2Vy | base64 -d && echo ""
```

Not a surprise right ? Do the same for the password

```bash
echo VjJ5dXJ5bUk4eldUZkZPYzFveDBOQ21iNHdSTl9Bd0dNZWpoNGFrU1hFblgtZTNZVTFlRWpR | base64 -d && echo ""
```

- **C2 - Connect to SSH to one of the node**

We are using the pod name `cluster1-dc1-rack1-sts-0`

```bash
kubectl -n cass-operator exec -it cluster1-dc1-rack1-sts-0 -- /bin/bash
```

- **C3 - Display status of your cluster**

```bash
nodetool status
```

- **C4 - Connect with CQLSH**

The operator makes a Kubernetes headless service available at `<clusterName>-<datacenterName>-service`. Any CQL client inside the Kubernetes cluster should be able to connect to `cluster1-dc1-service.cass-operator` and use the nodes in a round-robin fashion as contact points.

```bash
cqlsh cluster1-dc1-service.cass-operator -u cluster1-superuser -p V2yurymI8zWTfFOc1ox0NCmb4wRN_AwGMejh4akSXEnX-e3YU1eEjQ
```

- **C5 - Create the killrvideo keyspace**

```sql
CREATE KEYSPACE IF NOT EXISTS killrvideo 
WITH replication = {'class': 'NetworkTopologyStrategy', 'dc1': '3'}
AND durable_writes = true;
```

- **C6 - Create tables and populate Data**

```sql
CREATE TABLE IF NOT EXISTS users (
    userid     uuid,
    firstname  text,
    lastname   text,
    email      text,
    created_date timestamp,
    PRIMARY KEY (userid)
);

CREATE TABLE IF NOT EXISTS videos (
    videoid                uuid,
    userid                 uuid,
    name                   text,
    description            text,
    location               text,
    location_type          int,
    preview_image_location text,
    tags                   set<text>,
    added_date             timestamp,
    PRIMARY KEY (videoid)
);

INSERT INTO killrvideo.videos (videoid, name, description, added_date)
  VALUES(99999999-1111-1111-1111-111111111111, 'Best of Jeff Carpenter', 'A highlights reel of some of Jeff''s greatest vlogs', toTimestamp(now()));
INSERT INTO killrvideo.videos (videoid, name, description, added_date)
  VALUES(99999999-2222-2222-2222-222222222222, 'David''s Bloopers', 'A collection of David''s missteps and fleet-a-foot recoveries', toTimestamp(now()));
INSERT INTO killrvideo.videos (videoid, name, description, added_date)
  VALUES(99999999-3333-3333-3333-333333333333, 'Cedrick''s Out-takes', 'It may sound like he''s cursing, but it''s just that he''s speaking French',  toTimestamp(now()));

INSERT INTO killrvideo.users (userid, created_date, firstname, lastname, email)
  VALUES(11111111-1111-1111-1111-111111111111, toTimestamp(now()), 'Jeffrey', 'Carpenter', 'jc@datastax.com');
//INSERT INTO killrvideo.user_credentials (userid, email, password)
//  VALUES(11111111-1111-1111-1111-111111111111, 'jc@datastax.com', 'J3ffL0v3$C@ss@ndr@');
  
INSERT INTO killrvideo.users (userid, created_date, firstname, lastname, email)
  VALUES(22222222-2222-2222-2222-222222222222, toTimestamp(now()), 'Eric', 'Zietlow', 'ez@datastax.com');
//INSERT INTO killrvideo.user_credentials (userid, email, password)
//  VALUES(22222222-2222-2222-2222-222222222222, 'ez@datastax.com', 'C@ss@ndr@R0ck$');

INSERT INTO killrvideo.users (userid, created_date, firstname, lastname, email)
  VALUES(33333333-3333-3333-3333-333333333333, toTimestamp(now()), 'Cedrick', 'Lunven', 'cl@datastax.com');
//INSERT INTO killrvideo.user_credentials (userid, email, password)
//  VALUES(33333333-3333-3333-3333-333333333333, 'cl@datastax.com', 'Fr@nc3L0v3$C@ss@ndr@');
```

- **C7 - Display data**

```sql
// List infos from users
SELECT * from killrvideo.users;

// List infos from videos
SELECT * from killrvideo.videos;
```

## D - Scale up your cluster


- **D1 - Let's create a cluster with minimal configuration**

```bash
kubectl -n workshop-cassandra apply -f ./1-cassandra/12-cassandra-cluster-1nodes.yaml
```

- **D2 - Look at this file where we changed the size**

```bash
diff ./1-cassandra/12-cassandra-cluster-1nodes.yaml ./1-cassandra/12-cassandra-cluster-3nodes.yaml
```

- **D3 - Apply the new configuration**


```bash
kubectl -n workshop-cassandra apply -f ./1-cassandra/12-cassandra-cluster-3nodes.yaml
```

## E - Change Cassandra Configuration



## Congratulations your are set.







