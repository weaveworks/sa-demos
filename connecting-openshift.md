This procedure is tested and working for OCP 4.11 and WG 0.9.4-rc2
There is a general documentation on how to connect clusters thruogh a service account on our [documentation](https://docs.gitops.weave.works/docs/cluster-management/managing-existing-clusters/)


# This is a guide on how to connect OpenShift Clusters to WGE Demo environments

OpenShift is a bit special. It is locked down with security contexts and also restricted in terms of which users it allows pods to run as. 
If we want to deploy flux / connect an OpenShift cluster to Weave Gitops, [we have to prepare it](https://fluxcd.io/flux/use-cases/openshift/). This preparation could be turned into a special OpenShift
Bootstrap config for automation.

Lacking a sepcialized OpenShift bootstrap automation, follow these steps : 

Log into your OpenShift on the cli
```bash
$ oc login --token=... --server=https://api.lutz-rosa.p1ug.p1.openshiftapps.com:6443
```

Set the **nonroot** SCC for all controllers in the flux-system ns
```bash
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
```bash
$ CLUSTER=lutz-openshift-rosa
$ cd ~/git/demo2-repo
$ mkdir -p clusters/default/$CLUSTER/flux-system
$ touch clusters/default/$CLUSTER/flux-system/{gotk-components,gotk-sync,kustomization}.yaml
```

**_NOTE_** Using a user token based kubeconfig is not a good solution for long term automation and for adding OpenShift to WGE. The user token times out after 24h. 

**We want to create a service account for weave gitops to access the OpenShift cluster**

Create the flux-system namespace 
```
$ oc create ns flux-system
```

Create a new *weave-gitops* service account
```
$ oc create sa weave-gitops -n flux-system
```

Allow weave-gitops to do it's work (**not advised in production USE FINE GRAINED ACCESS CONTROL**)
```
$ oc policy add-role-to-user cluster-admin -z weave-gitops -n flux-system
$ oc create clusterrolebinding weave-gitops-binding --clusterrole=cluster-admin --serviceaccount=flux-system:weave-gitops
```

This will give you the API Token for the Service Account
```bash
$ KUBE_API_TOKEN=$(oc sa new-token weave-gitops -n flux-system)
```

We will need the CA Cert for the OpenShift cluster to be able to verify the connection. My OpenShift has been signed using LetsEncrpyt R3. We can download the certificate as txt ( same fmt as crt ) from [Letsencrypt](https://letsencrypt.org/certificates/). 
```
$ KUBE_CERT=weave-gitops-platform/openshift-extras/lets-encrypt-r3.pem
```

This guide helps to build a kubeconfig that uses the service account token

```
$ KUBE_API_EP='https://api.lutz-rosa.p1ug.p1.openshiftapps.com:6443'
$ CLUSTER=openshift-lutz-rosa
```

Let's clean our current kubeconfig again
```
$ mv ~/.kube/config ~/.kube/ocpconfig
```

The next commands will create a new kubeconfig with the KUBE_ vars that we have defined earlier.
```
kubectl config set-cluster "$CLUSTER" --server="$KUBE_API_EP" \
    --certificate-authority="$KUBE_CERT"  \
    --embed-certs=true
kubectl config set-credentials weave-gitops --token=$KUBE_API_TOKEN
kubectl config set-context $CLUSTER --cluster $CLUSTER --user weave-gitops
kubectl config use-context $CLUSTER \

```

Let's test this kubeconfig
```
$ oc get pods -A
```

Move the Service Account kubeconfig out of the way and restore your old kubeconfig.
```
$ mv $HOME/.kube/config $HOME/secrets/ocp-weave-gitops-kubeconfig
$ cp $HOME/.kube/oldconfig $HOME/.kube/config
```

Now we create a secret that holds this kubeconfig. We need a secret that holds a kubeconfig to connect our OpenShift cluster to Weave Gitops. We are defining the secret on the command line and load it into our management cluster manually. You could you a key management system for this as well. 
```
$ kubectl config use-context LutzAdm@wge-demo2.eu-west-3.eksctl.io
$ kubectl create secret generic openshift-lutz-rosa-kubeconfig --from-file=value=$HOME/secrets/ocp-weave-gitops-kubeconfig
```

Precreate the structure of the cluster directory
```
$ cd ~/git/demo2-repo
$ git pull
$ mkdir -p clusters/default/openshift-lutz-rosa/flux-system
$ touch clusters/default/openshift-lutz-rosa/flux-system/{gotk-components,gotk-sync,kustomization}.yaml
$ cat << EOF > clusters/default/openshift-lutz-rosa/flux-system/kustomization.yaml
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
                    \$patch: delete      
    target:
      kind: Deployment
      labelSelector: app.kubernetes.io/part-of=flux
EOF 


```

Connecting a cluster manually will not run put the bases kustomization into the cluster directory. We have to do this manually as well
```
cat << EOF > clusters/default/openshift-lutz-rosa/clusters-bases-kustomization.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  creationTimestamp: null
  name: clusters-bases-kustomization
  namespace: flux-system
spec:
  interval: 10m0s
  path: clusters/bases
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
EOF
git add clusters/default/openshift-lutz-rosa/ && git commit -m 'prime openshift' && git push

```


We create a GitOpsCluster object that will reference our kubeconfig secret. This references the default bootstrap configuration, but will not fail, as we have perpared the OpenShift specific setting beforehand. ( Security Context Constraints, Service Account, Service Account Token, Kubeconfig and kustomization.yaml patch )
```
cat << EOF > clusters/management/clusters/default/openshift-lutz-rosa.yaml
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
EOF
git add clusters/management/clusters/default/openshift-lutz-rosa.yaml
git pull && git commit -m 'add GitOpsCluster for OpenShift' && git push

```


# deleting a cluster that was connected this way

Delete the kubeconfig from the cluster, and remove the GitOpsCluster object 
```
kubectl delete secret openshift-lutz-rosa-kubeconfig
git rm clusters/management/clusters/default/openshift-lutz-rosa.yaml
git pull && git commit -m "deleting $CLUSTER" && git push
flux reconcile kustomization flux-system --with-source

```
