# Module 5: Federated endpoint identity

**Goal:** 
Configure a local cluster to pull endpoint data from a remote cluster. Federating endpoints allows teams to write policies in a local cluster that references/selects endpoints in remote clusters. This expands efficiency and self-service.

# Steps

**Cluster A**

**Create kubeconfig files**

- Create RBAC
```bash
kubectl apply -f \
https://downloads.tigera.io/ee/v3.15.1/manifests/federation-rem-rbac-kdd.yaml
```
- Apply the following manifest to create a service account
```bash
kubectl apply -f \
https://downloads.tigera.io/ee/v3.15.1/manifests/federation-remote-sa.yaml
```

- Create a secret for service Account manually.
| NOTE:This step is needed if your k8s version is 1.24 or above. From K8s 1.24, K8s won’t generate Secrets automatically for ServiceAccounts and need be created manually. 

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: tigera-federation-remote-cluster
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: "tigera-federation-remote-cluster"
EOF
```

- Create virables 
```bash
CLUSTER=cluster-a
YOUR_SERVICE_ACCOUNT_TOKEN=$(kubectl get secret -n kube-system tigera-federation-remote-cluster -o jsonpath='{.data.token}'|base64 --decode)
YOUR_CERTIFICATE_AUTHORITY_DATA=$(kubectl config view --flatten --minify -o jsonpath='{range .clusters[*]}{.cluster.certificate-authority-data}{"\n"}{end}')
YOUR_SERVER_ADDRESS=$(kubectl config view --flatten --minify -o jsonpath='{range .clusters[*]}{.cluster.server}{"\n"}{end}')
```
- Create kubeconfig file 
```bash
cat > ./$CLUSTER-kubeconfig <<'EOF'
apiVersion: v1
kind: Config
users:
- name: tigera-federation-remote-cluster
  user:
    token: YOUR_SERVICE_ACCOUNT_TOKEN
clusters:
- name: tigera-federation-remote-cluster
  cluster:
    certificate-authority-data: YOUR_CERTIFICATE_AUTHORITY_DATA
    server: YOUR_SERVER_ADDRESS
contexts:
- name: tigera-federation-remote-cluster-ctx
  context:
    cluster: tigera-federation-remote-cluster
    user: tigera-federation-remote-cluster
current-context: tigera-federation-remote-cluster-ctx
EOF
```
 
 - Update kubeconfig file 

```bash
sed -i  s,YOUR_SERVICE_ACCOUNT_TOKEN,$YOUR_SERVICE_ACCOUNT_TOKEN,g ./$CLUSTER-kubeconfig
sed -i  s,YOUR_CERTIFICATE_AUTHORITY_DATA,$YOUR_CERTIFICATE_AUTHORITY_DATA,g ./$CLUSTER-kubeconfig
sed -i  s,YOUR_SERVER_ADDRESS,$YOUR_SERVER_ADDRESS,g ./$CLUSTER-kubeconfig
```

- exchange kubeconfig files between sites 

**create secret for each site**
```bash
kubectl create secret generic remote-cluster-secret-cluster-b -n calico-system \
    --from-literal=datastoreType=kubernetes \
    --from-file=kubeconfig=cluster-b-kubeconfig
kubectl create secret generic remote-cluster-secret-cluster-c -n calico-system \
    --from-literal=datastoreType=kubernetes \
    --from-file=kubeconfig=cluster-c-kubeconfig
```

- Create access to secrets for clusters
```bash
kubectl create -f config/remote-cluster-configuration-rbac.yaml
```
-  Add remote cluster configurations
```bash
kubectl create -f config/remote-cluster-configuration-clusterB.yaml
kubectl create -f config/remote-cluster-configuration-clusterC.yaml
```

**Cluster B**

**Create kubeconfig files**

- Create RBAC
```bash
kubectl apply -f \
https://docs.tigera.io/getting-started/kubernetes/installation/federation-rem-rbac-kdd.yaml
```
- Apply the following manifest to create a service account
```bash
kubectl apply -f \
https://docs.tigera.io/getting-started/kubernetes/installation/federation-remote-sa.yaml
```
- Create virables 
```bash
CLUSTER=cluster-b
YOUR_SERVICE_ACCOUNT_TOKEN=$(kubectl get secret -n kube-system tigera-federation-remote-cluster -o jsonpath='{.data.token}'|base64 --decode)
YOUR_CERTIFICATE_AUTHORITY_DATA=$(kubectl config view --flatten --minify -o jsonpath='{range .clusters[*]}{.cluster.certificate-authority-data}{"\n"}{end}')
YOUR_SERVER_ADDRESS=$(kubectl config view --flatten --minify -o jsonpath='{range .clusters[*]}{.cluster.server}{"\n"}{end}')
```
- Create kubeconfig file 
```bash
cat > ./$CLUSTER-kubeconfig <<'EOF'
apiVersion: v1
kind: Config
users:
- name: tigera-federation-remote-cluster
  user:
    token: YOUR_SERVICE_ACCOUNT_TOKEN
clusters:
- name: tigera-federation-remote-cluster
  cluster:
    certificate-authority-data: YOUR_CERTIFICATE_AUTHORITY_DATA
    server: YOUR_SERVER_ADDRESS
contexts:
- name: tigera-federation-remote-cluster-ctx
  context:
    cluster: tigera-federation-remote-cluster
    user: tigera-federation-remote-cluster
current-context: tigera-federation-remote-cluster-ctx
EOF
```
 
 - Update kubeconfig file 

```bash
sed -i  s,YOUR_SERVICE_ACCOUNT_TOKEN,$YOUR_SERVICE_ACCOUNT_TOKEN,g ./$CLUSTER-kubeconfig
sed -i  s,YOUR_CERTIFICATE_AUTHORITY_DATA,$YOUR_CERTIFICATE_AUTHORITY_DATA,g ./$CLUSTER-kubeconfig
sed -i  s,YOUR_SERVER_ADDRESS,$YOUR_SERVER_ADDRESS,g ./$CLUSTER-kubeconfig
```

- exchange kubeconfig files between sites 

**create secret for each site**
```bash
kubectl create secret generic remote-cluster-secret-cluster-a -n calico-system \
    --from-literal=datastoreType=kubernetes \
    --from-file=kubeconfig=cluster-a-kubeconfig
kubectl create secret generic remote-cluster-secret-cluster-c -n calico-system \
    --from-literal=datastoreType=kubernetes \
    --from-file=kubeconfig=cluster-c-kubeconfig
```

- Create access to secrets for clusters
```bash
kubectl create -f config/remote-cluster-configuration-rbac.yaml
```
-  Add remote cluster configurations
```bash
kubectl create -f config/remote-cluster-configuration-clusterA.yaml
kubectl create -f config/remote-cluster-configuration-clusterC.yaml
```

**Cluster C**

**Create kubeconfig files**

- Create RBAC
```bash
kubectl apply -f \
https://docs.tigera.io/getting-started/kubernetes/installation/federation-rem-rbac-kdd.yaml
```
- Apply the following manifest to create a service account
```bash
kubectl apply -f \
https://docs.tigera.io/getting-started/kubernetes/installation/federation-remote-sa.yaml
```
- Create virables 
```bash
CLUSTER=cluster-c
YOUR_SERVICE_ACCOUNT_TOKEN=$(kubectl get secret -n kube-system tigera-federation-remote-cluster -o jsonpath='{.data.token}'|base64 --decode)
YOUR_CERTIFICATE_AUTHORITY_DATA=$(kubectl config view --flatten --minify -o jsonpath='{range .clusters[*]}{.cluster.certificate-authority-data}{"\n"}{end}')
YOUR_SERVER_ADDRESS=$(kubectl config view --flatten --minify -o jsonpath='{range .clusters[*]}{.cluster.server}{"\n"}{end}')
```
- Create kubeconfig file 
```bash
cat > ./$CLUSTER-kubeconfig <<'EOF'
apiVersion: v1
kind: Config
users:
- name: tigera-federation-remote-cluster
  user:
    token: YOUR_SERVICE_ACCOUNT_TOKEN
clusters:
- name: tigera-federation-remote-cluster
  cluster:
    certificate-authority-data: YOUR_CERTIFICATE_AUTHORITY_DATA
    server: YOUR_SERVER_ADDRESS
contexts:
- name: tigera-federation-remote-cluster-ctx
  context:
    cluster: tigera-federation-remote-cluster
    user: tigera-federation-remote-cluster
current-context: tigera-federation-remote-cluster-ctx
EOF
```
 
 - Update kubeconfig file 

```bash
sed -i  s,YOUR_SERVICE_ACCOUNT_TOKEN,$YOUR_SERVICE_ACCOUNT_TOKEN,g ./$CLUSTER-kubeconfig
sed -i  s,YOUR_CERTIFICATE_AUTHORITY_DATA,$YOUR_CERTIFICATE_AUTHORITY_DATA,g ./$CLUSTER-kubeconfig
sed -i  s,YOUR_SERVER_ADDRESS,$YOUR_SERVER_ADDRESS,g ./$CLUSTER-kubeconfig
```

- exchange kubeconfig files between sites 

**create secret for each site**
```bash
kubectl create secret generic remote-cluster-secret-cluster-a -n calico-system \
    --from-literal=datastoreType=kubernetes \
    --from-file=kubeconfig=cluster-a-kubeconfig
kubectl create secret generic remote-cluster-secret-cluster-b -n calico-system \
    --from-literal=datastoreType=kubernetes \
    --from-file=kubeconfig=cluster-b-kubeconfig
```

- Create access to secrets for clusters
```bash
kubectl create -f config/remote-cluster-configuration-rbac.yaml
```
-  Add remote cluster configurations
```bash
kubectl create -f config/remote-cluster-configuration-clusterA.yaml
kubectl create -f config/remote-cluster-configuration-clusterB.yaml
```


