**work in progress**

# Tenancy demo guide

Imagine you are a service provider that has a platform team that creates and manages clusters. These clusters are installed for different clustomers. A customer or tenant can have 1 or many clusters. 

* The platform team wants visiblity across the entire estate. 
* The tenants should only see their clusters.

There are 2 ways to accomodate the needs 

A) Shared WGE UI
B) Dedicated WGE UIs per platform team and per tenant

There is the choice if you want to allow tenants to create clusters or not. This is up to the platform team / business to decide.

# Tenancy as released in the product

Is a general tooling that was demonstrated to slice a cluster into shards that are given to tenants. The shards are fenced of with policy and RBAC settings. Kevin told me that the general model can acommodate other tenancy scenarios as well, but I did not investigate that.

[Tenancy documentation](https://docs.gitops.weave.works/docs/enterprise/multi-tenancy/)
- the documentation is work in progress.
- the model is to generate a tenancy.yaml file and use gitops cli to expand that and put the expanded tenancy-definition into the management and the leave cluster git repostiory locations.
- Your leaf cluster will need the have the policy-agent installed and will need to have the admission controller in enforcing mode.

[Tenancy presentation](https://docs.google.com/presentation/d/1deuqVlg2UEhda9_z3FVW61xWBENCWP-c0VLMk7VUCh4/edit#slide=id.gf40d68bd3d_4_0)

While the tenancy model is very flexible, it does not deliver out of the box capabilities that I need for the clusters per tenant model. It is currently more geared towards a slicing up one cluster into multiple tenants scenario.

The clusters per tenant model is something that we could get to with RBAC currently. There is a bug however that prevents us from restricting access to the default ns for tenants. ( Simon is working on this, but it might land in v0.9.7  )

## Test A - with a devteam1 tenant through RBAC

There are a few questions that need answers. What do you want tenants to be able to do?

* Create Clusters? - Yes
* Access Clusters? - Yes
* Access only certain namespaces in leaf cluster ( this is what currently works with the tenany model )

I'm starting by creating a **devteam1** tenant. 

One option is to create namespace and my-pat secret manually
```
kubectl create ns devteam1
kubectl create secret generic my-pat --from-literal GITHUB_TOKEN=$GITHUB_TOKEN -n devteam1
```

We can create cluster and separate them per namespace. Thus we need all CapiTemplate, GitOpsCluster, and bootstrap objects in the devteam1 namespace. These are essentially copies of the defaut ns ones. I've put them into **devteam1** subdirs in [capi-templates](https://github.com/weavegitops/demo3-repo/tree/main/weave-gitops-platform/capi-templates/devteam1) and [capi-profiles](https://github.com/weavegitops/demo3-repo/tree/main/weave-gitops-platform/capi-profiles/devteam1).

This allows the admin user to create a cluster in the devteam1 namespace. 

The result was that my user did not have access to the newly created cluster that was put into the devteam1 namespace. At least it did not show in the Cluster list. 

![Screenshot from 2022-10-14 10-10-23](https://user-images.githubusercontent.com/2788194/195808620-f1a4bd6f-a8bb-441b-84e6-1c50d57fcce6.png)

## B - using multiple UIs to get cluster level tenancy.

If we can't get RBAC and our UI working in a way that gives us tenancy, we could run multiple UIs.

### Model 1 - Managment Cluster + Leaf Cluster with WGE tenant UI

One Management UI for the Admins
- Cluster creating on Managment cluster
- Observablity on Tenant UI ( WGEE )

One WGE UI per Tenant ( installed in 1st tenant cluster )

Both Admin and Tenant UI have the same leaf cluster connected.
- They would need to be connected with the same git repository config, but that is only a flux thing...
- This should work as it is essentially only a GitOpsCluster and a kubeconfig secret that is required.
- Only issue is when editing the cluster as the path to git can't be configured currently.

Can we restrict access to the clusters correctly?
- RBAC access can be but on leaf clusters for the tenants needing access
- GitOpsCluster Objects can be placed in the default ns on the leaf management cluster.

* Create EKS cluster
- don't use the default capa profile. It will create a cluster where the Nodes are on the private VPN only. You can't access the WGE UI easily there.
- I've created an EKS cluster with eksctl

Issue with the Token from the kubeconfig. Switching to EC2 based cluster instead.

* Activate Cluster issuer
- Connect the cluster to demo3 management for visiblity and bootstrap.

1. Create the kubeconfig secret for devteam-eks2 in demo3 ( is already there if EC2 cluster ist capi provisioned )
```
mv ~/.kube/config ~/.kube/oldconfig
aws-login
aws eks update-kubeconfig --region eu-central-1 --name devteam-eks2
mv ~/.kube/config ~/.kube/devteam-eks2.config
mv ~/.kube/oldconfig ~/.kube/config
kubectl config use-context LutzAdm@wge-demo3.eu-west-1.eksctl.io
kubectl create secret generic devteam-eks2-kubeconfig --from-file=value=$HOME/.kube/devteam-eks2.config
```

2. Build and commit GitOpsCluster Object 
```
cd $HOME/git/demo3-repo
cat <<EOF > clusters/management/clusters/defaulls
t/devteam-eks2.yaml

apiVersion: gitops.weave.works/v1alpha1
kind: GitopsCluster
metadata:
  name: devteam-eks2
  namespace: default
  # Signals that this cluster should be bootstrapped.
  labels:
    weave.works/capi: bootstrap
spec:
  secretRef:
    name: devteam-eks2-kubeconfig
EOF
git add clusters/management/clusters/default/devteam-eks2.yaml
git commit -m 'connect devteam-eks2'
git push
```

* Deploy WGE to leaf cluster

- deactivte capi in helm chart
- add dex client secret
```
ku 1 create secret generic dex-client-credentials \
      --from-literal=clientID=weave-gitops-enterprise \
      --from-literal=clientSecret=mySecretIsNotSave135 \
      -n flux-system
```
- add weave-gitops-enterprise-credentials secret
```
ENTITLEMENTFILE="$HOME/.wge/new-entitlement.yaml"
ku 1 create -f "$ENTITLEMENTFILE"
```
- create auth secret for admin user "Demo2SecretPw!"
```
ku 1 create secret generic cluster-user-auth --namespace flux-system --from-literal=username=wego-admin --from-literal=password='$2a$10$fU9b05.5l/xsS24bfWBnOeU5Q.gBFR6ROtYlN2PMJ5foicpaBxusC'
```
- add route53 dns entry
- enable OIDC in Google
- add dex client secret manually

* Expose UI
* Configure OICD
* Add GitOps Cluster

**NOTE: see the interal WGE deployment and [template work on this](https://github.com/weaveworks/corp-fleet/blob/main/templates/bases/eks_machinedeployment_wge_external_repo/eks_cluster.yaml)**
