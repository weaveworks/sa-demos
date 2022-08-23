# Application Demos

Application Demos currently require that you add Git Repository objects before you can use the UI to add the kustomization

If you want to be able to add your application to any cluster. It is best to have it available as part of the base kustomization. All leaf clusters get the cluster base kustomization assinged per default. 

Put your GitRepository Object into ./clusters/bases/git-repos

### Appplication-Podinfo-UAT branch example 

The following example is using branches as a staging strategy and has a uat (User Acceptance Test) branch. 
- note that branches as staging strategy are not the recommended approach, but we have customers with that strategy
  - the better strategy is using subdirectories and tags or separate repositories for stages

```bash
$ cd ~/git/demo3-repo/clusters/bases/git-repos
```
```bash
$ flux create source git application-podinfo-uat \
       --url=ssh://git@github.com/weavegitops/application-podinfo.git \
       --branch=uat \
       --interval=1m \
       --private-key-file=$HOME/.ssh/id_ecdsa \
       -n flux-system \
       --export > application-podinfo-uat.yaml
```
```bash
$ git add application-podinfo-uat.yaml
```
```bash
$ git commit -m "add application-podinfo-uat.yaml' && git push
```
