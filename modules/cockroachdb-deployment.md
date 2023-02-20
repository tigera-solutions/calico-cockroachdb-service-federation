# Step: Deploy CockroachDB

In each region we need to prepare each cluster for CockroachDB in a multi-region configuration. In each cluster we need to create a `namespace` and update the coredns configuration via a `configmap`.

## Region 1

In the first cluster run the commands below. These will create the namespace and take a backup of the existing `configmap`. 

```
kubectl create namespace us-east-1
kubectl -n kube-system get configmap coredns -o yaml > cockroach/region1-configmap-back.yaml
```
The `configmap` needs to be updated to forward DNS requests for cockroachdb pods in other clusters to DNS running in the same cluster. An example of this is below.

```
    us-east-2.svc.cluster.local:53 {       # <---- Modify
        log
        errors
        ready
        cache 10
        forward . 172.21.0.10 {      # <---- Modify
        }
    }
    us-west-1.svc.cluster.local:53 {       # <---- Modify
        log
        errors
        ready
        cache 10
        forward . 172.22.0.10 {      # <---- Modify
        }
    }
```

Once updated to forward the DNS to the correct coredns service in the target cluster you can replace the existing `configmap` and restart coredns.

```
kubectl -n kube-system replace -f region1-coredns-configmap.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
kubectl -n kube-system describe configmap coredns
```

## Region 2

Repeat for the seconf cluster.

```
kubectl create namespace us-east-2
kubectl -n kube-system get configmap coredns -o yaml > cockroach/region2-configmap-back.yaml
```
Replace and restart coredns.

```
kubectl -n kube-system replace -f region2-coredns-configmap.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
kubectl -n kube-system describe configmap coredns
```

## Region 3

Finally the the third cluster.

```
kubectl create namespace us-west-1
kubectl -n kube-system get configmap coredns -o yaml > cockroach/region3-configmap-back.yaml
```

Replace and restart coredns.

```
kubectl -n kube-system replace -f region3-coredns-configmap.yaml
kubectl delete pod --namespace kube-system --selector k8s-app=kube-dns
kubectl -n kube-system describe configmap coredns
```

## Deploy Cockroach DB

Cockroach has an internal CA to create and manage certificates. On a instance running the cockroach binary.

Create root certificate and client. This certificate needs to be installed on all three clusters, this allows all the pods to be able to communicate with each other.

Create some directories to store the certificates.
```
mkdir certs my-safe-directory
```

Create the CA certificate.
```
cockroach cert create-ca \
--certs-dir=certs \
--ca-key=my-safe-directory/ca.key
```
Create the client certificate. 
```
cockroach cert create-client \
root \
--certs-dir=certs \
--ca-key=my-safe-directory/ca.key
```

The contents `certs` directory now has to be stored as a secret in each cluster to be mounted in to the file system of the Cockroachdb pods.

## Region 1

In region one create the secret.
```
kubectl create secret \
generic cockroachdb.client.root \
--from-file=certs \
--namespace us-east-1
```

## Region 2

Ensure that you have the same `certs` directory as the first cluster, then create the secret. 
```
kubectl create secret \
generic cockroachdb.client.root \
--from-file=certs \
--namespace us-east-2
```

## Region 3

Ensure that you have the same `certs` directory as the first cluster, then create the secret. 

```
kubectl create secret \
generic cockroachdb.client.root \
--from-file=certs \
--namespace us-west-1
```

## Region 1 - us-east-1

Now node certificates need to be created for the nodes in each region. These need to be unique for each region as the certificates will contain Subject Alternate Names (SAN's) for the pods which are specific to each region.

Run this command where cockroach binary is installed.
```
cockroach cert create-node \
localhost 127.0.0.1 \
cockroachdb-public \
cockroachdb-public.us-east-1 \
cockroachdb-public.us-east-1.svc.cluster.local \
"*.cockroachdb" \
"*.cockroachdb.us-east-1" \
"*.cockroachdb.us-east-1.svc.cluster.local" \
--certs-dir=certs \
--ca-key=my-safe-directory/ca.key
```

Create the secret in the first cluster.
```
kubectl create secret \
generic cockroachdb.node \
--from-file=certs \
--namespace us-east-1
```

As we are running the commands on a single instance we need to remove the the first node certificate. I production you would want to keep these safe...
```
rm certs/node.crt
rm certs/node.key
```

## Region 2 - us-east-2

Create the node certificate for second region.

```
cockroach cert create-node \
localhost 127.0.0.1 \
cockroachdb-public \
cockroachdb-public.us-east-2 \
cockroachdb-public.us-east-2.svc.cluster.local \
"*.cockroachdb" \
"*.cockroachdb.us-east-2" \
"*.cockroachdb.us-east-2.svc.cluster.local" \
--certs-dir=certs \
--ca-key=my-safe-directory/ca.key
```

Make sure you have the full contents of the `certs` directory again on the second cluster including the node certificate and key.
Create the secret.

```
kubectl create secret \
generic cockroachdb.node \
--from-file=certs \
--namespace us-east-2
```

Remove the node cert.

```
rm certs/node.crt
rm certs/node.key
```

## Region 3 - us-west-1

Create the node certificate for third region.

```
cockroach cert create-node \
localhost 127.0.0.1 \
cockroachdb-public \
cockroachdb-public.us-west-1 \
cockroachdb-public.us-west-1.svc.cluster.local \
"*.cockroachdb" \
"*.cockroachdb.us-west-1" \
"*.cockroachdb.us-west-1.svc.cluster.local" \
--certs-dir=certs \
--ca-key=my-safe-directory/ca.key
```

Make sure you have the full contents of the `certs` directory again on the third cluster including the node certificate and key.
Create the secret.

```
kubectl create secret \
generic cockroachdb.node \
--from-file=certs \
--namespace us-west-1
```

Remove node certificate.

```
rm certs/node.crt
rm certs/node.key
```

## Apply StatefulSet in each region.

Apply the cockroach kubernetes manifeast on each of the clusters.

```
kubectl -n us-east-1 apply -f cockroach/region1-cockroachdb-statefulset-secure.yaml
kubectl -n us-east-2 apply -f cockroach/region2-cockroachdb-statefulset-secure.yaml
kubectl -n us-west-1 apply -f cockroach/region3-cockroachdb-statefulset-secure.yaml
```

## Region 1

Initialize the databases. Run this command that will deploy a pod and initialize the cluster.

```
kubectl exec \
--namespace us-east-1 \
-it cockroachdb-0 \
-- /cockroach/cockroach init \
--certs-dir=/cockroach/cockroach-certs
```

## Each Region

CHeck the pods are running and ready.

```
kubectl get pods --namespace us-east-1
kubectl get pods --namespace us-east-2
kubectl get pods --namespace us-west-1
```

## Region 1

Create a user in the database to enable access to the admin UI.
Create a pod
```
kubectl create -f https://raw.githubusercontent.com/cockroachdb/cockroach/master/cloud/kubernetes/multiregion/client-secure.yaml --namespace us-east-1
```
Connect to the pod..

```
kubectl exec -it cockroachdb-client-secure -n us-east-1 -- ./cockroach sql --certs-dir=/cockroach-certs --host=cockroachdb-public
```
Create the user.
```
CREATE USER craig WITH PASSWORD 'cockroach';
GRANT admin TO craig;
\q
```
