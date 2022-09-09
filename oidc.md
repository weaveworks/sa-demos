# work in progress

# This documents describes the deployment of OIDC in our demo environments

We need to use Dex to configure OICD Logins for Weave Gitops as Google does not provide group information directly but through a 2nd call. Dex implements getting 
group information from OIDC as well.

We need 3 Secrets for Dex. Long term these should be in AWS Secrets Manager, 1st step is to define them directly on the the CLI and use kubectl to 
install them in the managment cluster. 

The json files for the following secrets are save in a 1P vault. "WGE SA Demos"

- dex-oauth-app-credentials are associated with the definition of the Web App (Dex) in Google.
- dex-google-sa is the service account that allows access to Google objects.

Download the json files from the 1P vault and create these two secrets in ns dex-system.
```
$ kubectl create ns dex-system
$ kubectl create secret generic dex-google-sa \
      --from-literal=adminEmail=lutz@weave.works \
      --from-file=corp-oidc.json=lutz-democenter-fdce2315e44f.json \
      -n dex-system
      
$ kubectl create secret generic dex-oauth-app-credentials \
      --from-literal=CLIENT_ID=716282251124-nsh1u1qtrodinkhaup290qak3vcbevk4.apps.googleusercontent.com\
      --from-literal=CLIENT_SECRET=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\
      -n dex-system
```

We do need a 3rd secret between Dex and Weave Gitops. This can be a generic random secret for the exchange between the two.
```
$ kubectl create secret generic dex-client-credentials --from-literal=clientSecret=mySecretIsNotSave135 -n dex-system
```

Now that the secrets are created, we can add the dex configuration. This is prepared in ~/git/demo2-repo/weave-gitops-platform/dex . We activate it by adding a kustomization to ~/git/demo2-repo/clusters/management/

```
$ cat << EOF > 25-dex.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: dex 
  namespace: flux-system
spec:
  interval: 2m0s
  path: ./weave-gitops-platform/dex
  prune: true
  wait: true
  timeout: 5m
  sourceRef:
    kind: GitRepository
    name: flux-system
EOF
$ flux reconcile kustomization flux-system --with-source
```

Now check that dex runs happily
```
$ k get pods -n dex-system
NAME                        READY   STATUS    RESTARTS   AGE
cm-acme-http-solver-8g2kw   1/1     Running   0          26m
dex-79dcfbdfc8-4dk77        1/1     Running   0          72s
```

Let's check the Ingress :
```
$ k get ingress -n dex-system
NAME                        CLASS    HOSTS                       ADDRESS                                                                   PORTS     AGE
cm-acme-http-solver-lwlmb   <none>   dex-demo2.weavegitops.com   a0f00ce38d0d44dce815d4ad71be9a34-1159077847.eu-west-3.elb.amazonaws.com   80        27m
```

We don't have external-dns yet, so we need to update Route53 manually.
dex                         <none>   dex-demo2.weavegitops.com   a0f00ce38d0d44dce815d4ad71be9a34-1159077847.eu-west-3.elb.amazonaws.com   80, 443   27m

![Screenshot from 2022-09-09 19-32-06](https://user-images.githubusercontent.com/2788194/189416540-31855887-2a69-436b-b280-b45674dd9f54.png)

Check that dns and ssl is working by opening https://dex-demo2.weavegitop.com in a browser window.
![Screenshot from 2022-09-09 20-11-02](https://user-images.githubusercontent.com/2788194/189416703-325496b1-067e-4f88-b123-f607f02bc3bb.png)

Create the static client secret in flux-system for Weave Gitops to consume :
```
$ kubectl create secret generic dex-client-credentials --from-literal=clientSecret=mySecretIsNotSave135 -n flux-system
```


