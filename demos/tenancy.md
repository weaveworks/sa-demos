**work in progress**

[Tenancy documentation](https://docs.gitops.weave.works/docs/enterprise/multi-tenancy/)
- the documentation is work in progress.
- the model is to generate a tenancy.yaml file and use gitops cli to expand that and put the expanded tenancy-definition into the management and the leave cluster git repostiory locations.
- Your leaf cluster will need the have the policy-agent installed and will need to have the admission controller in enforcing mode.

[Tenancy presentation](https://docs.google.com/presentation/d/1deuqVlg2UEhda9_z3FVW61xWBENCWP-c0VLMk7VUCh4/edit#slide=id.gf40d68bd3d_4_0)

While the tenancy model is very flexible, it does not deliver out of the box capabilities that I need for the clusters per tenant model. It is currently more geared towards a slicing up one cluster into multiple tenants scenario.

The clusters per tenant model is something that we could get to with RBAC currently. There is a bug however that prevents us from restricting access to the default ns for tenants. ( Simon is working on this, but it might land in v0.9.7  )

## Test A - with a devteam1 tenant

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

This allows the admin user to create a cluster in the devteam1 namespace. The CAPITemplate needs to reside in the default ns currently. If we put it into devteam1 it will not be visible in the UI. (bug)

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


```kind: ClusterRole
metadata:
  labels:
    app.kubernetes.io/instance: weave-gitops-enterprise
    rbac.authorization.k8s.io/aggregate-to-limited-users-reader: "true"
  name: devteam1-gitopsclusters-reader
  resourceVersion: "157739226"
  uid: 4a491f23-09ce-4c6e-843a-84870eea0e1f
rules:
- apiGroups:
  - gitops.weave.works
  resources:
  - gitopsclusters
  resourceNames:
  - "devteam*"
  verbs:
  - get
  - watch
  - list
```

And a [limited-users Cluster Role](https://github.com/weavegitops/demo3-repo/blob/main/weave-gitops-platform/weave-gitops/limited-users-role.yaml) that I created by taking the gitops reader role und ajusting a few access right down so that they only match certain objects

## Test C - using multiple UIs to get cluster level tenancy.

If we can't get RBAC and our UI working in a way that gives us tenancy, we could run multiple UIs.

### Model 1 - Managment Cluster + Leaf Clusters with Weave GitOps UI

One Management UI for the Admins
- Cluster creating on Managment cluster
- Observablity on Tenant UI ( Weave Gitops Core )

--> there is no cluster view in Tenant UI, just application
--> only one cluster per tenant UI

This is tested and it works.
Update using Weave GitOps: v0.10.0 UI is empty. Am I missing RBAC? Using LM clusters :
Certificate is not ready, as https solver can't work, and dns solver is not set up correctly for this. I would need a way to manipulate Route 53 from my leaf cluster or do it manually.

--> Try again using EKS instead of LM for now.
* Create EKS cluster
* Activate Cluster issuer


### Model 2 - Management Cluster + Leaf Clusters with WGE UI

One WGE UI for Admins

One WGE UI per Tenant ( installed in 1st tenant cluster )

? Can both Admin and Tenant UI have the same leaf cluster connected?
- They would need to be connected with the same git repository config, but that is only a flux thing...
- This should work as it is essentially only a GitOpsCluster and a kubeconfig secret that is required.

? Can we restrict access to the clusters correctly?
- RBAC access can be but on leaf clusters for the tenants needing access
- GitOpsCluster Objects can be placed in the default ns on the leaf management cluster.

* Create EKS cluster
* Activate Cluster issuer
* Deploy WGE to leaf cluster
* Expose UI
* Configure OICD
* Add GitOps Cluster

