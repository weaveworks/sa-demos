GitRepository or HelmRepository Objects need to be added to the management and to the leaf cluster if you want to be able to use the UI to provising Kustomizations or helm charts.

There is 2 locations for this :

Management Cluster (we might want to change this location / rename it) :
```bash
$ ls ~/git/demo2-repo/weave-gitops-platform/capi-profiles
app-charts.yaml  isolavent-charts.yaml  podinfo-app-gitrepo.yaml  profile-repo.yaml
```

Base Directory for all leaf clusters
```bash
$ ls ~/git/demo2-repo/clusters/bases/sources/
app-charts.yaml  isolavent-charts.yaml  podinfo-app-gitrepo.yaml
```

Make sure to include the profile annotation if you add a regular helm repository that has charts without the profile labels. 
Otherwise you will not see charts in the UI :
```bash
$ cat ~/git/demo2-repo/clusters/bases/sources/isolavent-charts.yaml 
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: isovalent-charts
  namespace: flux-system
  annotations:
    weave.works/profiles: "true"
spec:
  interval: 1m
  url: https://helm.isovalent.com/
```
