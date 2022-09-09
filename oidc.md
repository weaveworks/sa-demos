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
$ kubectl create secret generic dex-google-sa --from-file=value=lutz-democenter-fdce2315e44f.json -n dex-system
$ kubectl create secret generic dex-oauth-app-credentials --from-file=value=client_secret_716282251124-nsh1u1qtrodinkhaup290qak3vcbevk4.apps.googleusercontent.com.json -n dex-system
```

We do need a 3rd secret between Dex and Weave Gitops. This can be a generic random secret for the exchange between the two.
```
$ kubectl create secret generic dex-client-credentials --from-literal=value=mySecretIsNotSave135 -n dex-system
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


Warning  Failed     10s (x7 over 74s)  kubelet            Error: couldn't find key CLIENT_ID in Secret dex-system/dex-oauth-app-credentials
