# work in progress

see [cluster-update doc](https://eksctl.io/usage/cluster-upgrade/)


0. Make sure to use the latest eksctl
1. Update the control plane version
```
$ eksctl upgrade cluster --name wge-demo2 --region eu-west-3
2022-09-01 14:34:26 [ℹ]  (plan) would upgrade cluster "wge-demo2" control plane from current version "1.20" to "1.21"
2022-09-01 14:34:27 [ℹ]  re-building cluster stack "eksctl-wge-demo2-cluster"
2022-09-01 14:34:27 [✔]  all resources in cluster stack "eksctl-wge-demo2-cluster" are up-to-date
2022-09-01 14:34:27 [ℹ]  checking security group configuration for all nodegroups
2022-09-01 14:34:27 [ℹ]  all nodegroups have up-to-date cloudformation templates
2022-09-01 14:34:27 [!]  no changes were applied, run again with '--approve' to apply the changes
lutz@dellbook ~/git/cx-presales/lutzdemo/scripts (main) 
$ eksctl upgrade cluster --name wge-demo2 --region eu-west-3 --approve
2022-09-01 14:34:38 [ℹ]  will upgrade cluster "wge-demo2" control plane from current version "1.20" to "1.21"
```

2. replace each of the nodegroups by creating a new one and deleting the old one
3. update default add-ons
* kube-proxy
* aws-node
* core-dns
