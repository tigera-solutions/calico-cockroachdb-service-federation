Ensure pod-pod communication between clusters A (pod cidr 10.0.0.0/16), B (pod cidr 10.1.0.0/16) and C (pod cidr 10.2.0.0/16).

**Make calico use IPIP when reaching remote pod network**
Create following in each cluster (example is given for cluster A):

```bash
apiVersion: crd.projectcalico.org/v1
kind: IPPool
metadata:
  name: B-ippool
spec:
  cidr: 172.22.0.0/21  # podCIDR used in cluser B
  ipipMode: Always   # crucial - configures Bird to use IPIP for learned routes within given CIDR
  natOutgoing: false
  disabled: true          # 'disabled' option has effect for IP assignment purposes only
```
