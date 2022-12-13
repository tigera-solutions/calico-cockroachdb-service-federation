In this section we will install kubeadm cluster in each region with calico CNI.
you can follow the normal steps to install the kubeadm cluster on ubuntu https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

I used bash script to install kubadm with one command line as described in the folloing link: 
https://github.com/JosephYostos/kubeadm-installation

make sure to edir the pod-network-cidr & service-cidr Default on each cluster

Pod IP setup
Cluster A: 192.168.0.0/18
Cluster B: 192.168.64.0/18
Cluster C: 192.168.128.0/18

Service IP setup 
Cluster A: 172.20.0.0/16
Cluster B: 172.21.0.0/16
Cluster C: 172.22.0.0/16
