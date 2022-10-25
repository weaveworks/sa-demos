# sa-demos
This repository is dealing with the Weave GitOps demos and demo environments. Please use it to file and update issues that you are seeing. You can find the official demo guides here.

# Demo Status

## Demo2 is the STABLE environment, do not modify, this will be used for Kubecon NA.
https://demo2.weavegitops.com
- running v0.9.5
- EKS is working use **aws-eks-dev** to create EKS clusters
- LM provisioning is working with the **lm-edge** template
- Pipelines feature flag: On
- TerraformUI feature flag: Off

## Demo3 is now the new developing environment.
https://demo3.weavegitops.com
- running v0.9.5
-  EKS is working use **aws-eks-dev** to create EKS clusters
- Liquid Metal is working use **lm-edge** to create clusters
- Pipelines feature flag: On
- TerraformUI feature flag: Off


### TODO
- create clusters for app-promotion demo
- install TF-controller

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

## Demo Instructions

Read the demo instructions fully and run a practice before presenting to a customer.

- [Cluster Deployment with CAPI in WGE](https://github.com/weaveworks/sa-demos/blob/main/demos/cluster_capi.md)
- [Deploy Applications in WGE](https://github.com/weaveworks/sa-demos/blob/main/demos/applications.md)
- [Application Promotion and Policy Checks using Flux and WGE](https://github.com/weaveworks/sa-demos/blob/main/demos/application_promotion.md)
- [Policy and Trusted Delivery](https://github.com/weaveworks/sa-demos/blob/main/demos/policy-trusted_delivery.md)
- [Progressive Delivery with WGE](https://github.com/weaveworks/sa-demos/blob/main/demos/progressive-delivery-demo.md)

Please also keep these instructions up to date for SAs to reference easily.
