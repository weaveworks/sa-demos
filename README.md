# sa-demos
This repository is dealing with the Weave GitOps demos and demo environments. Please use it to file and update issues that you are seeing. You can also put your demo guides and demo scripts here.

# Demo Status

## Demo3 is currently the stable environment
https://demo3.weavegitops.com
- running 0.9.3 
- Liquid Metal is working use **lm-edge** to create clusters
- EKS is working use **aws-eks-dev** to create EKS clusters
- Add Application flow is broken - don't try to put things in flux-system

## Demo2 is currently unstable
https://demo2.weavegitops.com
- running v0.9.4-rc.2
- EKS is working use **aws-eks-dev** to create EKS clusters
- LM provisioning is working with the **lm-edge** template
- broken Sources for leaf cluster listings

## Accessing the Management Clusters

You can find the IAM Credentials in 1P : 

![Screenshot from 2022-09-07 09-43-27](https://user-images.githubusercontent.com/2788194/188821862-4ca062e0-bd38-4839-8186-257cf625215b.png)

Put these in your environment. You can add the kubeconfig data for demo2 to your existing kubeconfig with this command :
```
$ aws eks --region eu-west-3 update-kubeconfig --name wge-demo2 
```

For demo3 use 
```
$ aws eks --region eu-west-1 update-kubeconfig --name wge-demo3
```
