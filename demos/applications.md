# Work in progress

## Basic App Demo
This demo is designed to show how simple it is to deploy applications to Kubernetes using Weave GitOps (Flux). It is therefore a 
golden path and tries to hide complex functions in favour of simplicity. It should be used when the audience are still at the early stages of
moving towards cloud native and we are not looking to overwhelm with
art of the possible.

## Prerequisites
- **Git Repository source** - can be manually created with `flux create source git podinfo --url https://github.com/weavegitops/podinfo-app --branch main --export`
then checked into an appropriate location to be reconciled to the cluster.

## Demo steps

### Initial deployment

1. Login to the management cluster using OIDC or cluster admin
1. Navigate to the applications page
1. Click add application
1. Fill out the form as follows:
  1.  cluster: management
  1.  source: GitRepository/podinfo
  1.  kustomization name: podinfo
  1.  kustomization namespace: (*leave as flux-system*)
  1.  target namespace: (*leave blank*)
  1.  path: \/
1. Authenticate with GitHub
1. Create and merge pull request
1. Navigate back to Applications page
1. Sync flux-system with source
1. Show podinfo kustomization and walk through the tabs

### Successful update
1. Click hyperlink to source repo
1. Push change to update backend deployment with new image tag, i.e. 6.1.0
1. Navigate back to podinfo kustomization
1. Sync with source
1. Watch success

---

# Application Demos

Adding an Application to a Leaf Cluster.

There are 2 types of Sources and then there is possibly Secrets. As Secrets make things more complicated, we stick to public Sources or Sources we do have the key for on the leaf and on the management cluster already.

## GitRepository as Source

A) management cluster as source**

**Management cluster as Application Source**
PRO: 
- no secret needs to be added
- full control over source
CON: 
- not realistic as app sources are usually outside of the management repository

**public repository as source**
PRO: 
- no secrets need to be added
- separate repo is more realistic
- option to manipulate code easily
CON: 
- visible to all

We can create public repositories on the **weavegitops** org easily.

For quick demo use [Podinfo App](https://github.com/weavegitops/podinfo-app) 

We can define sources for public repos like this :
```bash
cd ~/git/demo2-repo/
flux create source git podinfo-app --url=https://github.com/weavegitops/podinfo-app.git --branch=main --interval=5m -n flux-system --export >> clusters/bases/sources/podinfo-app-gitrepo.yaml
flux create source git podinfo-app --url=https://github.com/weavegitops/podinfo-app.git --branch=main --interval=5m -n flux-system --export >> weave-gitops-platform/capi-profiles/podinfo-app-gitrepo.yaml
git add weave-gitops-platform/capi-profiles/podinfo-app-gitrepo.yaml clusters/bases/sources/podinfo-app-gitrepo.yaml
git pull
git commit -m podinfo-gitrepo-yaml
git push
```

We have added the GitRepository Object to the [management cluster](https://github.com/weavegitops/demo2-repo/blob/main/weave-gitops-platform/capi-profiles/podinfo-app-gitrepo.yaml) and to all [leaf clusters](https://github.com/weavegitops/demo2-repo/blob/main/clusters/bases/sources/podinfo-app-gitrepo.yaml)

This makes it available in the Add Application UI.


**private repository as source**


## Helm Repository as Source

We have the **app-charts** public repository for helm.

-------


Application Demos currently require that you have a GitRepository object and the required secret before you can use the UI to add the kustomization.

## Demo with Helm Chart and the app in the mangement repostiory

Pro :
- no need to deal with secrets when the app is in the management repo
- no need to add a GitRepository as flux-system bootstrap has done this for us

Con : 
- no separate repo

There is a podinfo helm chart in the management repo already. We can use that to deploy and application. 

Here is how you can deploy that podinfo app with the UI : 

Let's modify the helm release.

This fails, but the flux falls back and we still have the app running.

Modify back to a version that does exist. 


## Demo using the CLI to add the GitRepository and the Secret to the Leaf Cluster

Pro : 
- nothing else required

Con : 
- the GitRepository is not in git
- you need to use the flux command line
- you need to have the leaf cluster kubeconfig on the cli
- the secret is created from the cli in the cluster
- there is no central secrets management

**_NOTE:_** there is currently a bug for EKS clusters, that will give you a kubeconfig that is only valid for 10 min. You can work around it by downloading the kubeconfig on the cli with 'aws eks update-kubeconfig --region eu-central-1 --name myekscluster

Download the Leaf cluster kubeconfig and make its context default
```
$ export KUBECONFIG=$HOME/Downloads/my-kubeconfig
```
or try 
```
$ export KUBECONFIG=$HOME/Downloads/$(ls -rt ~/Downloads/ | tail -1)
```

Make sure you are running a recent flux version
```
$ flux version
flux: v0.32.0
helm-controller: v0.21.0
kustomize-controller: v0.25.0
notification-controller: v0.23.5
source-controller: v0.24.4
```

Create a new GitRepository & secret for your Application Repository
```
$ flux create source git lutzlm6-podinfo --https://github.com/weavegitops/podinfo-deploy --branch=master --interval=1m 
```

If you want to be able to add your application to any cluster. It is best to have it available as part of the base kustomization. All leaf clusters get the cluster base kustomization assinged per default. 

Put your GitRepository Object into ./clusters/bases/git-repos

### Appplication-Podinfo-UAT branch example 

Stages could be implemented through branches in the Git repository, but we do advise agains this. The better way is to use directories and tags as the standard podinfo example does. This has the benefit of requiring only one GitRepository source that can be used to deploy all stages of the application.

```bash
$ cd ~/git/demo3-repo/clusters/bases/git-repos
```

Using --export allows us to sidestep kubectl and kubeconfig configuration here. We just let the leaf cluster flux pick this up. 
We are reusing the flux-system secret here that were set up as part of the flux bootstrap process. This application repository is in the same GitHub org and the ssh_key in the flux-system secret will allow access.

```bash
$ flux create source git application-podinfo \
       --url=ssh://git@github.com:weavegitops/application-podinfo.git \
       --secret-ref flux-system \
       --branch=main \
       --interval=1m \
       --private-key-file=$HOME/.ssh/id_ecdsa \
       -n flux-system \
       --export > application-podinfo-gitrepo.yaml
```

Add, commit and push to git.

```bash
$ git add application-podinfo-gitrepo.yaml && git commit -m 'add application-podinfo-gitrepo.yaml' && git push
```

You can wait for the leaf cluster flux-system kustomization to pick this up, or hit the 'sync' button in the WG UI.

## Adding the Application in the WG UI

You can now go to the WG UI Applications -> Add Application and fill in the required values : 
![Screenshot from 2022-08-23 12-40-23](https://user-images.githubusercontent.com/2788194/186138591-3f2ea82c-f4d6-4189-aa3d-489dbd3fca37.png)

Create and Merge the pull request. To deploy the podinfo app to the cluster.

