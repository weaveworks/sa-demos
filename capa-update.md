## not finished

# How to update the AWS capa-controller and AWS IAM stack 

Authentication to the management clusters works with AWS IAM. I'm using LutzAdm IAM account for this. You can find the Keys in 1Password under "WGE SA Demo Env EKS IAM Secrets - LutzAdm" .

The management clusters are installed with the "weaveworks-cx" AWS account. Go to Gmail -> Google Apps -> Amazon Web Services to access the AWS UI.

We need to update 2 cli tools, the IAM Cloud Formation Stack and redo the capa-controller on the demo management EKS cluster.

Download the latest clusterctl and clusterawsadm tools :
...

### Update CloudFormation IAM Stack

There is a CloudFormation Stack that is installed into the AWS account. While the CloudFormation Stack is regional, the IAM accounts are global. The stack is currently installed into "eu-west-1". The default name of this CF Stack is 'cluster-api-provider-aws-sigs-k8s-io'.

You can try updating the stack with : 
```
$ clusterawsadm bootstrap iam create-cloudformation-stack --region eu-west-1
```

If this does not work, delete the Stack from the UI and install it again fresh from the CLI.


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
