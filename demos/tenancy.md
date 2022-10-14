**work in progress**

[Tenancy documentation](https://docs.gitops.weave.works/docs/enterprise/multi-tenancy/)
- the documentation is work in progress.
- the model is to generate a tenancy.yaml file and use gitops cli to expand that and put the expanded tenancy-definition into the management and the leave cluster git repostiory locations.
- Your leaf cluster will need the have the policy-agent installed and will need to have the admission controller in enforcing mode.

[Tenancy presentation](https://docs.google.com/presentation/d/1deuqVlg2UEhda9_z3FVW61xWBENCWP-c0VLMk7VUCh4/edit#slide=id.gf40d68bd3d_4_0)

While the tenancy model is very flexible, it does not deliver out of the box capabilities that I need for the clusters per tenant model. It is currently more geared towards a slicing up one cluster into multiple tenants scenario.

The clusters per tenant model is something that we could get to with RBAC currently. There is a bug however that prevents us from restricting access to the default ns for tenants. ( file bug )

My next try is to 


## Test A - with a devteam1 tenant

There are a few questions that need answers. What do you want tenants to be able to do?

* Create Clusters? - Yes
* Access Clusters? - Yes
* Access only certain namespaces in leaf cluster?

I'm starting by creating a **devteam1** tenant. 

One option is to create namespace and my-pat secret manually
```
kubectl create ns devteam1
kubectl create secret generic my-pat --from-literal GITHUB_TOKEN=$GITHUB_TOKEN -n devteam1
```

The capability to create clusters is granted on a namespace access control. Thus we need all CapiTemplate, GitOpsCluster, and bootstrap objects in the devteam1 namespace. These are essentially copies of the defaut ns ones. I've put them into **devteam1** subdirs in [capi-templates](https://github.com/weavegitops/demo3-repo/tree/main/weave-gitops-platform/capi-templates/devteam1) and [capi-profiles](https://github.com/weavegitops/demo3-repo/tree/main/weave-gitops-platform/capi-profiles/devteam1).

The result was that my user did not have access to the newly created cluster that was put into the devteam1 namespace. At least it did not show in the Cluster list. 

The UI expects to have access to GitOps Templates and GitOps clusters in the default ns.

![Screenshot from 2022-10-14 10-10-23](https://user-images.githubusercontent.com/2788194/195808620-f1a4bd6f-a8bb-441b-84e6-1c50d57fcce6.png)

## Test B - restrict access to clusters in the default ns with RBAC

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: limited-users-clusterrole
subjects:
- kind: Group
  name: "team-demo-limited-users@weave.works"
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: limited-users
  apiGroup: rbac.authorization.k8s.io
```

And a [limited-users Cluster Role](https://github.com/weavegitops/demo3-repo/blob/main/weave-gitops-platform/weave-gitops/limited-users-role.yaml) that I created by taking the gitops reader role und ajusting a few access right down so that they only match certain objects

