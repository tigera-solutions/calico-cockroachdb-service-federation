# Module 3: Enabling across clusters pod-to-pod communication


**Goal:**

Ensure pod-pod communication between clusters A (pod cidr 10.0.0.0/16), B (pod cidr 10.1.0.0/16) and C (pod cidr 10.2.0.0/16).

# Steps

**Cluster A:**

- Create ip-ip ippools with the subnet of the other sites
```bash
kubectl apply -f config/fed-ippool-clusterA-config.yaml
```
- Configure a node to be a route reflector with cluster ID 244.0.0.1. 
**replace my-node with your master node hostname**
```bash
calicoctl patch node my-node -p '{"spec": {"bgp": {"routeReflectorClusterID": "244.0.0.1"}}}'
```
- Label this node to indicate that it is a route reflector. 
**replace my-node with your master node hostname**
```bash
kubectl label node my-node route-reflector=true
```
- configure route reflector nodes to peer with each other
```bash
kubectl apply -f config/BGPPeer_with_route_reflector.yaml
```
- create BGPConfiguration
```bash
kubectl apply -f config/BGPConfig_clusterA.yaml
```
- Create BGPPeer in each node (as number and peerip of the other site) ip address of the master
**Before creating BGPPeer, add IP of the remote RR**
```bash
kubectl apply -f config/BGPPeer_remote_rr_clusterA.yaml
```
- Make calico accept IPIP traffic
```bash
kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"externalNodesList":["10.1.1.0/24","10.2.1.0/24"]}}'
```

**Cluster B:**

- create ip-ip ippoolswith the subnet of the other sites
```bash
kubectl apply -f config/fed-ippool-clusterB-config.yaml
```
- configure a node to be a route reflector with cluster ID 244.0.0.2, 
**replace my-node with your master node hostname**
```bash
calicoctl patch node my-node -p '{"spec": {"bgp": {"routeReflectorClusterID": "244.0.0.2"}}}'
```
- Label this node to indicate that it is a route reflector. 
**replace my-node with your master node hostname**
```bash
kubectl label node my-node route-reflector=true
```
- configure route reflector nodes to peer with each other
```bash
kubectl apply -f config/BGPPeer_with_route_reflector.yaml
```
- create BGPConfiguration
```bash
kubectl apply -f config/BGPConfig_clusterB.yaml
```
- Create BGPPeer in each node (as number and peerip of the other site) ip address of the master  
**Before creating BGPPeer, add IP of the remote RR**
```bash
kubectl apply -f config/BGPPeer_remote_rr_clusterB.yaml
```
- Make calico accept IPIP traffic
```bash
kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"externalNodesList":["10.0.1.0/24","10.2.1.0/24"]}}'
```
**Cluster C:**

- create ip-ip ippoolswith the subnet of the other sites
```bash
kubectl apply -f config/fed-ippool-clusterC-config.yaml
```
- configure a node to be a route reflector with cluster ID 244.0.0.3, 
**replace my-node with your master node hostname**
```bash
calicoctl patch node my-node -p '{"spec": {"bgp": {"routeReflectorClusterID": "244.0.0.3"}}}'
```
- Label this node to indicate that it is a route reflector. 
**replace my-node with your master node hostname**
```bash
kubectl label node my-node route-reflector=true
```
- configure route reflector nodes to peer with each other
```bash
kubectl apply -f config/BGPPeer_with_route_reflector.yaml
```
- create BGPConfiguration
```bash
kubectl apply -f config/BGPConfig_clusterC.yaml
```
- Create BGPPeer in each node (as number and peerip of the other site) ip address of the master
**Before creating BGPPeer, add IP of the remote RR**
```bash
kubectl apply -f config/BGPPeer_remote_rr_clusterC.yaml
```
- Make calico accept IPIP traffic
```bash
kubectl patch felixconfiguration default --type='merge' -p '{"spec":{"externalNodesList":["10.0.1.0/24","10.1.1.0/24"]}}'
```

