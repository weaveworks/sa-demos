# work in progress

# This is a guide on how to connect OpenShift Clusters to WGE Demo environments

OpenShift is a bit special. It is locked down with security contexts and also restricted in terms of which users it allows pods to run as. 
If we want to deploy flux / connect an OpenShift cluster to Weave Gitops, [we have to prepare it](https://fluxcd.io/flux/use-cases/openshift/). This preparation could be turned into a special OpenShift
Bootstrap config for automation.

Lacking the sepcialized OpenShift Bootstrap automation, follow these steps.

Save your ~/.kube/config out of the way before you start :
```
$ mv ~/.kube/config ~/.kube/oldconfig
```

Log into your OpenShift on the cli
```
$ oc login --token=... --server=https://api.lutz-rosa.p1ug.p1.openshiftapps.com:6443
```

Set the **nonroot** SCC for all controllers in the flux-system ns
```
NS="flux-system"
oc adm policy add-scc-to-user nonroot system:serviceaccount:$NS:kustomize-controller
oc adm policy add-scc-to-user nonroot system:serviceaccount:$NS:helm-controller
oc adm policy add-scc-to-user nonroot system:serviceaccount:$NS:source-controller
oc adm policy add-scc-to-user nonroot system:serviceaccount:$NS:notification-controller
oc adm policy add-scc-to-user nonroot system:serviceaccount:$NS:image-automation-controller
oc adm policy add-scc-to-user nonroot system:serviceaccount:$NS:image-reflector-controller
oc adm policy add-scc-to-user nonroot system:serviceaccount:$NS:tf-controller
```

We will need to prepare to flux-system directory in the management repo. Go to the checkout of the demoX-repo :
```
$ cd ~/git/demo2-repo
$ mkdir clusters/default/
```

**_NOTE_** Using a user token is not a good solution as this kubeconfig holds a token that will timeout in 24h 

**We want to create a service account for weave gitops to access the OpenShift cluster**

Create a new *weave-gitops* service account
```
$ oc create sa weave-gitops
```

Allow weave-gitops to do it's work (**not advised in production USE FINE GRAINED ACCESS CONTROL**)
```
$ oc policy add-role-to-user cluster-admin -z weave-gitops
```

Let's clean our current kubeconfig again
```
$ mv ~/.kube/config ~/.kube/ocpconfig
```

This will give you the API Token for the Service Account
```
$ KUBE_API_TOKEN=$(oc sa get-token weave-gitops)
```

We will need the CA Cert for the OpenShift cluster to be able to verify the connection. My OpenShift has been signed using LetsEncrpyt R3. We can download the certificate as txt ( same fmt as crt ) from [Letsencrpy](https://letsencrypt.org/certificates/). 
```
$ KUBE_CERT=$HOME/git/demo2-repo/weave-gitops-platform/openshift-extras/lets-encrypt-r3.crt

This guide helps to build a kubeconfig that uses the service account token

```
$ KUBE_API_EP='https://api.lutz-rosa.p1ug.p1.openshiftapps.com:6443'
$ CLUSTER=openshift-lutz-rosa

kubectl config set-cluster $OPENSHIFT --server=$KUBE_API_EP \ 
    --certificate-authority=$KUBE_CERT  \
    --embed-certs=true
kubectl config set-credentials weave-gitops --token=$KUBE_API_TOKEN
kubectl config set-context $CLUSTER --cluster $CLUSTER --user weave-gitops
kubectl config use-context $CLUSTER
```

Let's test this kubeconfig



Now we create a secret that holds this kubeconfig. We need a secret that holds a kubeconfig to connect our OpenShift cluster to Weave Gitops. We are defining the secret on the command line and load it into our management cluster manually. You could you a key management system for this as well. 
```
$ kubectl create secret generic demo-01-kubeconfig --from-file=value=$HOME/.kube/config
```

Precreate the structure of the cluster directory
```
$ cd ~/git/demo2-repo
$ git pull
$ mkdir -p clusters/default/openshift-lutz-rosa/flux-system
$ touch clusters/default/openshift-lutz-rosa/flux-system/{gotk-components,gotk-sync,kustomization}.yaml
$ cat << EOF > clusters/openshift-lutz-rosa/flux-system/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - gotk-components.yaml
  - gotk-sync.yaml
patches:
  - patch: |
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: all
      spec:
        template:
          spec:
            containers:
              - name: manager
                securityContext:
                  runAsUser: 65534
                  seccompProfile:
                    $patch: delete      
    target:
      kind: Deployment
      labelSelector: app.kubernetes.io/part-of=flux
EOF 
$ git add clusters/default/openshift-lutz-rosa/
$ git commit -m 'prime openshift'
$ git push
```

We create a GitOpsCluster object that will reference our kubeconfig secret
```
$ cat << EOF > $HOME/git/demo2-repo/clusters/management/clusters/default/openshift-lutz-rosa.yaml
apiVersion: gitops.weave.works/v1alpha1
kind: GitopsCluster
metadata:
  name: openshift-lutz-rosa
  namespace: default
  # Signals that this cluster should be bootstrapped.
  labels:
    weave.works/capi: bootstrap
spec:
  secretRef:
    name: openshift-lutz-rosa-kubeconfig
```
