# DocumentDB Kubernetes Operator Guide

Learn how to run and manage DocumentDB on Kubernetes using the DocumentDB Kubernetes Operator. This quickstart guide will walk you through the steps to install the operator, deploy a DocumentDB cluster, connect to it, and perform basic operations.

## Prerequisites

- [Helm](https://helm.sh/docs/intro/install/) installed.
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) installed.
- A local Kubernetes cluster such as [minikube](https://minikube.sigs.k8s.io/docs/start/), or [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation) installed. You are free to use any other Kubernetes cluster, but that's not a requirement for this quickstart.

## Start a local Kubernetes cluster

If you are using `minikube`, use the following command:

```sh
minikube start
```

If you are using `kind`, use the following command:

```sh
kind create cluster
```

## Install `cert-manager`

[cert-manager](https://cert-manager.io/docs/) is used to manage TLS certificates for the DocumentDB cluster.

> If you already have `cert-manager` installed, you can skip this step.

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true
```

Verify that `cert-manager` is installed correctly:

```sh
kubectl get pods -n cert-manager
```

Output:

```text
NAMESPACE           NAME                                            READY   STATUS    RESTARTS
cert-manager        cert-manager-6794b8d569-d7lwd                   1/1     Running   0
cert-manager        cert-manager-cainjector-7f69cd69f7-pd9bc        1/1     Running   0          
cert-manager        cert-manager-webhook-6cc5dccc4b-7jmrh           1/1     Running   0          
```

## Install `documentdb-operator` using the Helm chart

> The DocumentDB operator utilizes the [CloudNativePG operator](https://cloudnative-pg.io/docs/) behind the scenes, and installs it in the `cnpg-system` namespace. At this point, it is assumed that the CloudNativePG operator is **not** pre-installed in your cluster.

Use the following command to install the DocumentDB operator:

```sh
helm install documentdb-operator oci://ghcr.io/microsoft/documentdb-operator --namespace documentdb-operator --create-namespace
```

This will install the operator in the `documentdb-operator` namespace. Verify that it is running:

```sh
kubectl get deployment -n documentdb-operator
```

Output:

```text
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
documentdb-operator   1/1     1            1           113s
```

You should also see the DocumentDB operator CRDs installed in the cluster:

```sh
kubectl get crd | grep documentdb
```

Output:

```text
documentdbs.db.microsoft.com
```

## Store DocumentDB credentials in Kubernetes Secret

Before deploying the DocumentDB cluster, create a Kubernetes secret to store the DocumentDB credentials. The sidecar injector plugin will automatically inject these credentials as environment variables into the DocumentDB gateway container.

Create the secret with your desired username and password:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: documentdb-preview-ns
---
# DocumentDB Credentials Secret
# 
# Login credentials:
# Username: k8s_secret_user
# Password: K8sSecret100
#
# Connect using mongosh (port-forward):
# mongosh 127.0.0.1:10260 -u k8s_secret_user -p K8sSecret100 --authenticationMechanism SCRAM-SHA-256 --tls --tlsAllowInvalidCertificates
#
# Connect using connection string (port-forward):
# mongosh "mongodb://k8s_secret_user:K8sSecret100@127.0.0.1:10260/?directConnection=true&authMechanism=SCRAM-SHA-256&tls=true&tlsAllowInvalidCertificates=true&replicaSet=rs0"
#

apiVersion: v1
kind: Secret
metadata:
  name: documentdb-credentials
  namespace: documentdb-preview-ns
type: Opaque
stringData:
  username: k8s_secret_user     
  password: K8sSecret100        
EOF
```

Verify the secret is created:

```sh
kubectl get secret documentdb-credentials -n documentdb-preview-ns
```

Output:

```text
NAME                     TYPE     DATA   AGE
documentdb-credentials   Opaque   2      10s
```

> **Note:** By default the operator expects a credentials secret named `documentdb-credentials` containing `username` and `password` keys. You can override the secret name by setting `spec.documentDbCredentialSecret` in your `DocumentDB` resource. Whatever name you configure (or the default) will be used by the sidecar injector to project the values as `USERNAME` and `PASSWORD` environment variables into the gateway sidecar container.

## Deploy a DocumentDB cluster

Create a single-node DocumentDB cluster:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: documentdb-preview-ns
---
apiVersion: db.microsoft.com/preview
kind: DocumentDB
metadata:
  name: documentdb-preview
  namespace: documentdb-preview-ns
spec:
  nodeCount: 1
  instancesPerNode: 1
  documentDbCredentialSecret: documentdb-credentials
  resource:
    storage:
      pvcSize: 10Gi
  exposeViaService:
    serviceType: ClusterIP
EOF
```

Wait for the DocumentDB cluster to be fully initialized. Verify that it is running:

```sh
kubectl get pods -n documentdb-preview-ns
```

Output:

```text
NAME                   READY   STATUS    RESTARTS   AGE
documentdb-preview-1   2/2     Running   0          26m
```

You can also check the DocumentDB CRD instance:

```sh
kubectl get DocumentDB -n documentdb-preview-ns
```

## Connect to the DocumentDB cluster

The DocumentDB `Pod` has the Gateway container running as a sidecar. The connection method depends on your service type. For local development you can configure the DocumentDB instance with the `ClusterIP` service type (as specified above), and use port forwarding to connect from your local machine:

**Step 1:** Set up port forwarding (keep this terminal open):

```sh
kubectl port-forward pod/documentdb-preview-1 10260:10260 -n documentdb-preview-ns
```

**Step 2:** In a **new terminal**, connect using `mongosh`:

```sh
# Traditional format
mongosh 127.0.0.1:10260 -u k8s_secret_user -p K8sSecret100 --authenticationMechanism SCRAM-SHA-256 --tls --tlsAllowInvalidCertificates

# Or connection string format
mongosh "mongodb://k8s_secret_user:K8sSecret100@127.0.0.1:10260/?directConnection=true&authMechanism=SCRAM-SHA-256&tls=true&tlsAllowInvalidCertificates=true&replicaSet=rs0"
```

Execute the following commands to create a database and a collection, and insert some documents:

```sh
use testdb

db.createCollection("test_collection")

db.test_collection.insertMany([
  { name: "Alice", age: 30 },
  { name: "Bob", age: 25 },
  { name: "Charlie", age: 35 }
])

db.test_collection.find()
```

Output:

```text
[direct: mongos] test> use testdb
switched to db testdb
[direct: mongos] testdb> db.createCollection("test_collection")
{ ok: 1 }
[direct: mongos] testdb> db.test_collection.insertMany([
...   { name: "Alice", age: 30 },
...   { name: "Bob", age: 25 },
...   { name: "Charlie", age: 35 }
... ])
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId('682c3b06491dc99ae02b3fed'),
    '1': ObjectId('682c3b06491dc99ae02b3fee'),
    '2': ObjectId('682c3b06491dc99ae02b3fef')
  }
}
[direct: mongos] testdb> db.test_collection.find()
[
  { _id: ObjectId('682c3b06491dc99ae02b3fed'), name: 'Alice', age: 30 },
  { _id: ObjectId('682c3b06491dc99ae02b3fee'), name: 'Bob', age: 25 },
  {
    _id: ObjectId('682c3b06491dc99ae02b3fef'),
    name: 'Charlie',
    age: 35
  }
]
```