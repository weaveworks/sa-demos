## not finished

# How to update the AWS capa-controller and AWS IAM stack 

Authentication to the management clusters works with AWS IAM. I'm using LutzAdm IAM account for this. You can find the Keys in 1Password under "WGE SA Demo Env EKS IAM Secrets - LutzAdm" .

The management clusters are installed with the "weaveworks-cx" AWS account. Go to Gmail -> Google Apps -> Amazon Web Services to access the AWS UI.

We need to update 2 cli tools, the IAM Cloud Formation Stack and redo the capa-controller on the demo management EKS cluster.

Download the latest clusterctl and clusterawsadm tools :
...


An update requires you to delete the old capa controller.
```
$ kubectl delete ns capa-system
```

Install the latest capa-controller
```
$ cd ~/git/cx-presales/lutzadm/scripts
$ ./
```

Install the aws cli. 

Download the kubeconfig.

Access the demo management cluster.


