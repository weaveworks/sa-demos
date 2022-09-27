# Documenting the state of tf-controller 

There are multiple ways to install tf-controller. We just went with the manual tf-controller installation as helm chart.

```
# Add tf-controller helm repository
helm repo add tf-controller https://weaveworks.github.io/tf-controller/

# Install tf-controller
helm upgrade -i tf-controller tf-controller/tf-controller \
    --namespace flux-system
```

As an alternative, we could create a Helmrelease and a Helmchart yaml and thus automate the setup with GitOps.

This is what we have done in the demo environments : 
```
$ cat ~/git/demo2-repo/clusters/management/44-tf-controller.yaml 
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: tf-controller
  namespace: flux-system
spec:
  interval: 2m0s
  path: ./weave-gitops-platform/tf-controller
  prune: true
  wait: true
  timeout: 5m
  sourceRef:
    kind: GitRepository
    name: flux-system

$ cat weave-gitops-platform/tf-controller/tf-controller.yaml 
---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: tf-controller
  namespace: flux-system
spec:
  interval: 1h0s
  url: https://weaveworks.github.io/tf-controller/
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: tf-controller
  namespace: flux-system
spec:
  chart:
    spec:
      chart: tf-controller
      sourceRef:
        kind: HelmRepository
        name: tf-controller
      version: '>=0.6.4'
  interval: 1h0s
  releaseName: tf-controller
  targetNamespace: flux-system
  install:
    crds: Create
  upgrade:
    crds: CreateReplace
  values:
    replicaCount: 1
    concurrency: 24
    resources:
      limits:
        cpu: 1000m
        memory: 2Gi
      requests:
        cpu: 400m
        memory: 64Mi
    caCertValidityDuration: 24h
    certRotationCheckFrequency: 30m
    image:
      tag: v0.12.0
    runner:
      image:
        tag: v0.12.0
```
