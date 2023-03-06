# sa-demos
This repository is dealing with the Weave GitOps demos and demo environments. Please use it to file and update issues that you are seeing. You can find the official demo guides here.

# Demo Overview

Consult [this table in notion](https://www.notion.so/weaveworks/e68deffcc9f24095b07607c45a7e13c9?v=bbe74a228384477eb4830869a8359fea) for a general overiew. 

# Demo Environment Status

## Demo2 is on the latest version and can be used for customer demos
https://demo2.weavegitops.com
- running v0.18.1
- EKS is working use **aws-eks-dev** to create clusters
- LM provisioning is working with the **lm-edge** template
- Demo clusters that must be deleted after the demo, just use: **demo-cluster-template** with the default values.
- Pipelines feature flag: On
- TerraformUI feature flag: On

## Demo3 is on the latest version and can be used for customer demos
https://demo3.weavegitops.com
- running v0.18.1
- EKS is working use **aws-eks-dev** to create clusters
- Liquid Metal is working use **lm-edge** to create clusters
- Demo clusters that must be deleted after the demo, just use: **demo-cluster-template** with the default values.
- Pipelines feature flag: On
- TerraformUI feature flag: On


### TODO
- document tenancy demo

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
