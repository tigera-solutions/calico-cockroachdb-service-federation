Ensure pod-pod communication between clusters A (pod cidr 10.0.0.0/16), B (pod cidr 10.1.0.0/16) and C (pod cidr 10.2.0.0/16).

**Make calico use IPIP when reaching remote pod network**

1- In each site create ip-ip ippoolswith the subnet of the other sites

Cluster A:
```bash
kubectl apply -f config/fed-ippool-clusterA-config.yaml
```


Cluster B:
```bash
kubectl apply -f config/fed-ippool-clusterB-config.yaml
```


Cluster C:
```bash
kubectl apply -f config/fed-ippool-clusterC-config.yaml
```

2- To configure a node to be a route reflector with cluster ID 244.0.0.1, run the following command.
NOTE: replace my-node with the controller node name

Cluster A:
```bash
calicoctl patch node my-node -p '{"spec": {"bgp": {"routeReflectorClusterID": "244.0.0.1"}}}'
```

Cluster B:
```bash
calicoctl patch node my-node -p '{"spec": {"bgp": {"routeReflectorClusterID": "244.0.0.2"}}}'
```

Cluster C:
```bash
calicoctl patch node my-node -p '{"spec": {"bgp": {"routeReflectorClusterID": "244.0.0.3"}}}'
```

3- label this node to indicate that it is a route reflector
