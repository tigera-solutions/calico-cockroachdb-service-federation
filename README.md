#  Calico Cluster-mesh for Multi-region DB

The goal of this example is to configure Calico's endpoint identity and service federation for kubernetes clusters using Calico CNI with an overlay pod network and residing in different VPCs.

## Learning Objectives

Organizations are adopting multi-region databases such as CockroachDB for different reasons like disaster recovery, regional data protection regulations or latency reduction. Unfortunately, Kubernetes single cluster architecture doesn't scale with the multi-region database deployment. Calico Cluster-mesh addresses this Kubernetes shortcoming by providing consistent networking and security policies for CockroachDB and similar architecture databases.
In this workshop we are going to focus on the following:
- Using calico Cluster-mesh to enable across cluster pod-to-pod communication.
- Configure a local cluster to pull endpoint data from a remote cluster.
- Configure local clusters to discover services across multiple clusters.

## Modules

- [Module 1: Provision AWS Infrastructure ](modules/provision-AWS-infra.md)
- [Module 2: Provision Kubernetes clusters with claico CNI](modules/provision-kubernetes-clusters.md)
- [Module 3: Enable across cluster pod-to-pod communication](modules/Enable-pod-to-pod-communication.md)
- [Module 4: Joining clusters to Calico Cloud](modules/Joining-clusters-to-Calico-Cloud.md)
- [Module 5: Federated endpoint identity](modules/federated-endpoint-identity.md)

