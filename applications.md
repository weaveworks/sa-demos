# Application Demos

Application Demos currently require that you add Git Repository objects before you can use the UI to add the kustomization

If you want to be able to add your application to any cluster. It is best to have it available as part of the base kustomization. All leaf clusters get the cluster base kustomization assinged per default. 

Put your GitRepository Object into ./clusters/bases/git-repos

e.g. 
```bash
$ cd ~/git/demo3-repo/clusters/bases/git-repos
$ flux create source git application-podinfo-uat --url=ssh://git@github.com/weavegitops/application-podinfo.git --branch=uat --interval=1m --private-key-file=$HOME/.ssh/id_ecdsa -n flux-system --export > application-podinfo-uat.yaml
```
