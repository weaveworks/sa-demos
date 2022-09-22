# sa-demos
This repository is dealing with the Weave GitOps demos and demo environments. Please use it to file and update issues that you are seeing. You can find the official demo guides here.

# Demo Status

## Demo3 is currently the stable environment this is our fall back
https://demo3.weavegitops.com
- running v0.9.4 
-  EKS is working use **aws-eks-dev** to create EKS clusters
- Liquid Metal is working use **lm-edge** to create clusters


## Demo2 is moving fast and might be unstable / this is where we try new stuff
https://demo2.weavegitops.com
- running v0.9.5
- EKS is working use **aws-eks-dev** to create EKS clusters
- LM provisioning is working with the **lm-edge** template

## Accessing the Management Clusters

You can find the IAM Credentials in 1P : 

![Screenshot from 2022-09-07 09-43-27](https://user-images.githubusercontent.com/2788194/188821862-4ca062e0-bd38-4839-8186-257cf625215b.png)

Put these in your environment. 

You can [download a kubeconfig](https://github.com/weaveworks/sa-demos/raw/main/kubeconfig/config) that has all 3 management clusters (demo[123]).

You can add the kubeconfig data for demo2 to your existing kubeconfig with this command :
```
$ aws eks --region eu-west-3 update-kubeconfig --name wge-demo2 
```

For demo3 use 
```
$ aws eks --region eu-west-1 update-kubeconfig --name wge-demo3
```
