Name:         dc1
Namespace:    cass-operator
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"cassandra.datastax.com/v1beta1","kind":"CassandraDatacenter","metadata":{"annotations":{},"name":"dc1","namespace":"cass-op...
API Version:  cassandra.datastax.com/v1beta1
Kind:         CassandraDatacenter
Metadata:
  Creation Timestamp:  2020-04-23T14:28:10Z
  Finalizers:
    finalizer.cassandra.datastax.com
  Generation:        2
  Resource Version:  21586
  Self Link:         /apis/cassandra.datastax.com/v1beta1/namespaces/cass-operator/cassandradatacenters/dc1
  UID:               eef9c664-a5aa-4979-a90d-3734a557c893
Spec:
  Cluster Name:  cluster1
  Config:
    Cassandra - Yaml:
      Authenticator:  org.apache.cassandra.auth.PasswordAuthenticator
      Authorizer:     org.apache.cassandra.auth.CassandraAuthorizer
      num_tokens:     8
      role_manager:   org.apache.cassandra.auth.CassandraRoleManager
    Jvm - Options:
      Additional - Jvm - Opts:
        -Dcassandra.system_distributed_replication_dc_names=dc1
        -Dcassandra.system_distributed_replication_per_dc=3
      initial_heap_size:  2G
      max_heap_size:      2G
  Management API Auth:
    Insecure:
  Racks:
    Name:  rack1
    Name:  rack2
    Name:  rack3
  Resources:
    Requests:
      Cpu:         1
      Memory:      4Gi
  Server Type:     cassandra
  Server Version:  3.11.6
  Size:            3
  Storage Config:
    Cassandra Data Volume Claim Spec:
      Access Modes:
        ReadWriteOnce
      Resources:
        Requests:
          Storage:         10Gi
      Storage Class Name:  server-storage
Status:
  Cassandra Operator Progress:  Updating
  Last Rolling Restart:         2020-04-23T14:28:10Z
  Last Server Node Started:     2020-04-23T14:30:46Z
  Node Statuses:
    cluster1-dc1-rack3-sts-0:
      Host ID:  defdcd08-9d05-4fe5-a0b0-86a6849fac9f
      Node IP:  10.244.5.4
Events:
  Type    Reason             Age    From           Message
  ----    ------             ----   ----           -------
  Normal  ScalingUpRack      2m56s  cass-operator  Scaling up rack rack3
  Normal  CreatedResource    2m56s  cass-operator  Created service cluster1-seed-service
  Normal  CreatedResource    2m56s  cass-operator  Created service cluster1-dc1-all-pods-service
  Normal  CreatedResource    2m56s  cass-operator  Created statefulset cluster1-dc1-rack1-sts
  Normal  CreatedResource    2m56s  cass-operator  Created statefulset cluster1-dc1-rack2-sts
  Normal  CreatedResource    2m56s  cass-operator  Created statefulset cluster1-dc1-rack3-sts
  Normal  ScalingUpRack      2m56s  cass-operator  Scaling up rack rack1
  Normal  ScalingUpRack      2m56s  cass-operator  Scaling up rack rack2
  Normal  CreatedResource    2m56s  cass-operator  Created service cluster1-dc1-service
  Normal  LabeledPodAsSeed   116s   cass-operator  Labeled pod a seed node cluster1-dc1-rack3-sts-0
  Normal  StartingCassandra  111s   cass-operator  Starting Cassandra for pod cluster1-dc1-rack3-sts-0
  Normal  StartedCassandra   83s    cass-operator  Started Cassandra for pod cluster1-dc1-rack3-sts-0
  Normal  StartingCassandra  83s    cass-operator  Starting Cassandra for pod cluster1-dc1-rack1-sts-0
  Normal  LabeledPodAsSeed   20s    cass-operator  Labeled as seed node pod cluster1-dc1-rack1-sts-0
  Normal  StartedCassandra   20s    cass-operator  Started Cassandra for pod cluster1-dc1-rack1-sts-0
  Normal  StartingCassandra  20s    cass-operator  Starting Cassandra for pod cluster1-dc1-rack2-sts-0