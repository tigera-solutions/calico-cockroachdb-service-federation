In the [github repository for CockroachDB](https://github.com/cockroachdb/cockroach) there is a Python script to help automate the deployment of CockroachDB across multiple regions. We need to update the Python script with the name of the Kubernetes contexts and regions that were created in the previous steps. 

```
contexts = { 'eastus': 'crdb-k3s-eastus', 'westus': 'crdb-k3s-westus', 'northeurpoe': 'crdb-k3s-northeurope',}
regions = { 'eastus': 'eastus', 'westus': 'westus', 'northeurope': 'northeurope',}
```

Once saved, the script can be run. This script deploys all the resources required across the three regions to run CockroachDB. Once this has been completed there is one final step to complete.

In each Kubernetes cluster there is a deployment of CoreDNS, this is responsible for name resolution in the Kubernetes cluster where it is deployed. It cannot however resolve names from other Kubernetes clusters without changes made to a ConfigMap containing the configuration of CoreDNS. This configuration can be updated to include forwarders for the two other clusters so pod in one cluster and resolve names of pods in another cluster. This is required for CockroachDB, all nodes in the cluster must be able to communicate with each other. Below is an example of the changes required in each Kubernetes cluster:

```
westus.svc.cluster.local:53 {       # <---- Modify
          log
          errors
          ready
          cache 10
          forward . IP1 IP2 IP3 {      # <---- Modify
          }
      }
      }
      northeurope.svc.cluster.local:53 {       # <---- Modify
          log
          errors
          ready
          cache 10
          forward . IP1 IP2 IP3 {      # <---- Modify
          }
      }

```
Apply the updated ConfigMaps to each of the clusters and the cluster should be established. 
