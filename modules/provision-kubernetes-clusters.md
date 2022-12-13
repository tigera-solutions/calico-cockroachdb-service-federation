In this section we will install kubeadm cluster in each region with calico CNI.
- floow the step in this document to install kubeadm cluster on each region with calico CNI
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
