In this section we will install kubeadm cluster in each region with calico CNI.
- follow the step in this document to install kubeadm cluster on each region with calico CNI
https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart

**Important**
To avoid any IPs overlabing we need to set a differnt pod network cidr and service cidr in each cluster
--pod-network-cidr
--service-cidr Default

make sure to edit the pod-network-cidr & service-cidr Default on each cluster

Pod IP setup
- Cluster A: 192.168.0.0/18
- Cluster B: 192.168.64.0/18
- Cluster C: 192.168.128.0/18

Service IP setup 
- Cluster A: 172.20.0.0/16
- Cluster B: 172.21.0.0/16
- Cluster C: 172.22.0.0/16


======

## Cluster A:

### server (K8 v1.25.6)

```bash
curl https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/kubeadm-master-a.sh |bash
```
install calico CNI (v3.24)

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/calico-custom-resources-a.yaml
```

### worker

```bash
curl https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/kubeadm-worker.sh |bash
```

## Cluster B:

### server (K8 v1.25.6)

```bash
curl https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/kubeadm-master-b.sh |bash
```
install calico CNI (v3.22)

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/calico-custom-resources-b.yaml
```

### worker

```bash
curl https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/kubeadm-worker.sh |bash
```

## Cluster C:

### server (K8 v1.25.6)

```bash
curl https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/kubeadm-master-c.sh |bash
```
install calico CNI (v3.22)

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.24.5/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/calico-custom-resources-c.yaml
```

### worker

```bash
curl https://raw.githubusercontent.com/tigera-solutions/calico-cockroachdb-service-federation/main/config/kubeadm-worker.sh |bash
```
