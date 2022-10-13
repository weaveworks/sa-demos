**work in progress**

[Tenancy documentation](https://docs.gitops.weave.works/docs/enterprise/multi-tenancy/)
- the documentation is work in progress.
- the model is to generate a tenancy.yaml file and use gitops cli to expand that and put the expanded tenancy-definition into the management and the leave cluster git repostiory locations.
- Your leaf cluster will need the have the policy-agent installed and will need to have the admission controller in enforcing mode.

[Tenancy presentation](https://docs.google.com/presentation/d/1deuqVlg2UEhda9_z3FVW61xWBENCWP-c0VLMk7VUCh4/edit#slide=id.gf40d68bd3d_4_0)

## Creating a new tenant

There are a few questions that need answers. What do you want tenants to be able to do?

* Create Clusters?
* Access Clusters?
* Access only certain namespaces in existing cluster?

I'm starting by creating a **devteam1** tenant. 


One option is to create namespace and my-pat secret manually
```
kubectl create ns devteam1
kubectl create secret generic my-pat --from-literal GITHUB_TOKEN=$GITHUB_TOKEN -n devteam1
```

The capability to create clusters is granted on a namespace access control. Thus we need all CapiTemplate, GitOpsCluster, and bootstrap objects in the devteam1 namespace. These are essentially copies of the defaut ns ones. I've put them into **devteam1** subdirs in [capi-templates](./weave-gitops-platform/capi-templates/devteam1) and [capi-profiles](weave-gitops-platform/capi-bootstrap/devteam1).
