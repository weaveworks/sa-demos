# Application Demos

Application Demos currently require that you add Git Repository objects before you can use the UI to add the kustomization

If you want to be able to add your application to any cluster. It is best to have it available as part of the base kustomization. All leaf clusters get the cluster base kustomization assinged per default. 

Put your GitRepository Object into ./clusters/bases/git-repos

### Appplication-Podinfo-UAT branch example 

Stages could be implemented through branches in the Git repository, but we do advise agains this. The better way is to use directories and tags as the standard podinfo example does. This has the benefit of requiring only one GitRepository source that can be used to deploy all stages of the application.

```bash
$ cd ~/git/demo3-repo/clusters/bases/git-repos
```

Using --export allows us to sidestep kubectl and kubeconfig configuration here. We just let the leaf cluster flux pick this up.

```bash
$ flux create source git application-podinfo \
       --url=ssh://git@github.com/weavegitops/application-podinfo.git \
       --branch=main \
       --interval=1m \
       --private-key-file=$HOME/.ssh/id_ecdsa \
       -n flux-system \
       --export > application-podinfo-gitrepo.yaml
```

Add, commit and push to git.

```bash
$ git add application-podinfo-gitrepo.yaml && git commit -m "add application-podinfo-gitrepo.yaml' && git push
```

You can wait for the leaf cluster flux-system kustomization to pick this up, or hit the 'sync' button in the WG UI.
