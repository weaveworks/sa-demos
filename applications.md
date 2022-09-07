# Work in progress

# Application Demos

Application Demos currently require that you have a GitRepository object and the required secret before you can use the UI to add the kustomization.

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

