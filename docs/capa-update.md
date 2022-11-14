## Official Route is blocked. Manual workaround implemented.

# How to update the AWS capa-controller and AWS IAM stack 

The management clusters are installed with the "weaveworks-cx" AWS account. Go to Gmail -> Google Apps -> Amazon Web Services to access the AWS UI.

We need to update 2 cli tools, the IAM Cloud Formation Stack and redo the capa-controller on the demo management EKS cluster.

Download the latest clusterctl and clusterawsadm tools :
...

## Authenticating to AWS on the CLI

We are using an interesting combination of AWS and Google. Our account managment is in Google and we do have a pass through authentication with AWS. You should install gsts on Linux or Mac to get CLI access. 

Note : On linux there is another default gsts tool that will conflict with what we need. This should be the right one : https://github.com/ruimarinho/gsts . You can use [aws-login](./tools/aws-login) to authenticate with gsts. This will open a Browser locally and you can authenticate with your Weaveworks login. There will be an AWS_PROFILE=sts env var afterwards. 

### Update CloudFormation IAM Stack

There is a CloudFormation Stack that is installed into the AWS account. While the CloudFormation Stack is regional, the IAM accounts are global. The stack is currently installed into "eu-west-1". The default name of this CF Stack is 'cluster-api-provider-aws-sigs-k8s-io'.

The next command needs to run with your weaveworks-cx account on the cli. You can try updating the stack with : 
```
$ clusterawsadm bootstrap iam create-cloudformation-stack --region eu-west-1
```

If this does not work, delete the Stack from the UI and install it again fresh from the CLI.

I was unable to use the EKS specific configuration
```
$ clusterawsadm bootstrap iam create-cloudformation-stack --config eks-bootstrap-config.yaml --region eu-west-1
Attempting to create AWS CloudFormation stack cluster-api-provider-aws-sigs-k8s-io
I0829 10:54:27.259486 1122583 service.go:68] AWS Cloudformation stack "cluster-api-provider-aws-sigs-k8s-io" already exists, updating
Error: failed to update AWS CloudFormation stack: ResourceNotReady: failed waiting for successful resource state
```

### Update the capa-controller

Authentication to the management clusters works with AWS IAM. I'm using LutzAdm IAM account for this. You can find the Keys in 1Password under "WGE SA Demo Env EKS IAM Secrets - LutzAdm" . You will need to remove the AWS_PROFILE var from your environment at this point :
```
$ unset AWS_PROFILE
```

Now set the vars for the IAM account.
...

An update requires you to delete the old capa controller.
```
$ kubectl delete ns capa-system
```

Install the latest capa-controller
```
$ cd ~/git/cx-presales/lutzadm/scripts
$ ./capa-eks-setup.sh ../demo2.conf
```

Install the aws cli. 

Download the kubeconfig.

Access the demo management cluster.

### Test deploying an EKS cluster

My nodes do restart and will not become ready. Investigation found :

```
$ kubectl get pods -A
kube-system   aws-node-l7zp9                            0/1     Running   5 (56s ago)   9m34s
kube-system   aws-node-rwwc8                            0/1     Running   5 (47s ago)   9m16s

$ kubectl logs aws-node-l7zp9 -n kube-system
{"level":"info","ts":"2022-08-29T08:25:55.626Z","caller":"entrypoint.sh","msg":"Checking for IPAM connectivity ... "}
{"level":"info","ts":"2022-08-29T08:25:57.666Z","caller":"entrypoint.sh","msg":"Retrying waiting for IPAM-D"}
{"level":"info","ts":"2022-08-29T08:25:59.679Z","caller":"entrypoint.sh","msg":"Retrying waiting for IPAM-D"}
...
```


This might be the same as:
https://github.com/aws/amazon-vpc-cni-k8s/issues/1847

This might fix it?

But we might be missing permission cni iam role?
https://docs.aws.amazon.com/eks/latest/userguide/cni-iam-role.html

So I tried :
```
$ eksctl utils associate-iam-oidc-provider --region=eu-central-1 --cluster=lutzeks7 --approve

$ eksctl create iamserviceaccount \
     --name aws-node \
     --namespace kube-system \
     --cluster lutzeks7 \
     --role-name "AmazonEKSVPCCNIRole" \
     --attach-policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy \
     --override-existing-serviceaccounts \
     --region eu-central-1 \
     --approve 

```

I can't access the cluster any more. Neighter the STS nor the LutzAdm way will give me access.

## Current state Sept-2022

It turned out that our account weaveworks-cx does hold items that were deployed with old IAM roles. Deleting the CloudFormation stack is not enough to fix this situation. We would need to clear out all objects using old roles and then start from scratch. Richard fixed permissions manually for us. The custerawsadm tool can be used to output the permissions template for the IAM CloudFormation stack. This gives you the input that is needed to manually create the missing IAM permissions.

It is planned to restrict the weaveworks-cx AWS account to SAs only. Starting 1-Oct-2022. We can clear out all old resources. 

There is no way back to the managed way. Richard Chase fixed our permissions manually before leaveing Weaveworks. We might need to switch to a fresh AWS account, or continue to manage permissions manually. There is a way to make clusterawsadm print out the template that is used to create the CloudFormation Stack that sets the permissions. That can then be used to manually check and adjust IAM permissions.






