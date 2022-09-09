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

**_NOTE_** This is not a good solution as this kubeconfig holds a token that will timeout in 24h 

We need a secret that holds a kubeconfig to connect our OpenShift cluster to Weave Gitops. We are defining the secret on the command line and load it into 
our management cluster manually. You could you a key management system for this as well. The above *oc login* has created a 
