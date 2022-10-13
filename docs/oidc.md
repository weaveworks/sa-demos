# How to deploy OIDC for Weaveworks demo environments

We need to use Dex to configure OICD Logins for Weave Gitops as Google does not provide group information directly but through a 2nd call. Dex implements getting 
group information from OIDC as well.

We need 3 Secrets for Dex. Long term these should be in AWS Secrets Manager, 1st step is to define them directly on the the CLI and use kubectl to 
install them in the managment cluster. 

The json files for the following secrets are save in a 1P vault. "WGE SA Demos"

- dex-oauth-app-credentials are associated with the definition of the Web App (Dex) in Google.
- dex-google-sa is the service account that allows access to Google objects.

Download the json files from the 1P vault and create these two secrets in ns dex-system.
```console
kubectl create ns dex-system
kubectl create secret generic dex-google-sa \
      --from-literal=adminEmail=oidc-auth-user@weave.works \
      --from-file=corp-oidc.json=lutz-democenter-fdce2315e44f.json \
      -n dex-system
      
kubectl create secret generic dex-oauth-app-credentials \
      --from-literal=CLIENT_ID=716282251124-nsh1u1qtrodinkhaup290qak3vcbevk4.apps.googleusercontent.com\
      --from-literal=CLIENT_SECRET=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\
      -n dex-system
```

We do need a 3rd secret between Dex and Weave Gitops. This can be a generic random secret for the exchange between the two.
```console
kubectl create secret generic dex-client-credentials \
    --from-literal=clientSecret=mySecretIsNotSave135 \
    -n dex-system
```

Now that the secrets are created, we can add the dex configuration. This is prepared in ~/git/demo2-repo/weave-gitops-platform/dex . We activate it by adding a kustomization to ~/git/demo2-repo/clusters/management/

```console
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
```console
$ k get pods -n dex-system
NAME                        READY   STATUS    RESTARTS   AGE
cm-acme-http-solver-8g2kw   1/1     Running   0          26m
dex-79dcfbdfc8-4dk77        1/1     Running   0          72s
```

Let's check the Ingress :
```console
$ k get ingress -n dex-system
NAME                        CLASS    HOSTS                       ADDRESS                                                                   PORTS     AGE
cm-acme-http-solver-lwlmb   <none>   dex-demo2.weavegitops.com   a0f00ce38d0d44dce815d4ad71be9a34-1159077847.eu-west-3.elb.amazonaws.com   80        27m
```

We don't have external-dns yet, so we need to update Route53 manually.

![Screenshot from 2022-09-09 19-32-06](https://user-images.githubusercontent.com/2788194/189416540-31855887-2a69-436b-b280-b45674dd9f54.png)

Check that dns and ssl is working by opening https://dex-demo2.weavegitop.com in a browser window.
![Screenshot from 2022-09-09 20-11-02](https://user-images.githubusercontent.com/2788194/189416703-325496b1-067e-4f88-b123-f607f02bc3bb.png)

Create the static client secret in flux-system for Weave Gitops to consume :
```console
kubectl create secret generic dex-client-credentials \
      --from-literal=clientID=weave-gitops-enterprise \
      --from-literal=clientSecret=mySecretIsNotSave135 \
      -n flux-system
```


Enable the [Admin SDK Api](https://console.cloud.google.com/apis/api/admin.googleapis.com/)


In Google Workspace Admin, add the Service Account´s client [Unique ID](https://console.cloud.google.com/iam-admin/serviceaccounts/details/116172021980214735487?project=lutz-democenter&supportedpurview=project) to the [Security → API controls → Domain-wide delegation](https://admin.google.com/ac/owl/domainwidedelegation) list.

![Screenshot from 2022-09-12 09-59-32](https://user-images.githubusercontent.com/25228551/189614463-ce93feeb-73f0-44bc-82e7-7482625b6042.png)
![Screenshot from 2022-09-12 10-00-36](https://user-images.githubusercontent.com/25228551/189614533-af4a203c-4550-4540-bf18-61c22a7426ab.png)

In Google Workspace Admin, use the OIDC auth user `oidc-auth-user@weave.works`: when assigned the `Groups Reader (BETA)` Role, it should have all necessary permissions to retrieve group memberships. Configure the GCP Service Account to impersonate this user.

Activate the OIDC in Weave-Gitops by adding the oidc block to the config section in ~/git/demo2-repo/weave-gitops-platform/weave-gitops.yaml
```yaml
...
    config:
      capi:
        repositoryURL: https://github.com/weavegitops/demo2-repo
      cluster-controller:
        enabled: true
      git:
        type: github
        hostname: "github.com"
      oidc:
        enabled: "true"
        clientCredentialsSecret: dex-client-credentials
        issuerURL: https://dex-demo2.weavegitops.com
        redirectURL: https://demo2.weavegitops.com/oauth2/callback
```


## Groups, User and Access

We do have the following groups now : 

**Groups**
- team-demo-admins@weave.works
- team-demo-viewers@weave.work
- team-demo-limited-users@weave.works


**Users**

We can use our regular users to access the demo environments. All Solution Architects are in the 

The passwords for the demo users are in 1P. You will need to login to a demo account before hitting the Login-with-OIDC button.

- demo-limited-user@weave.works
- demo-view-user@weave.works

**Admins**
Lutz, Darryl and Darren are currently group admins.

## RBAC configuration
There are two places for configuring RBAC currently. 
- for the management cluster : ` ~/git/demo2-repo/weave-gitops-platform/weave-gitops/read-all-role.yaml`
- for all leaf clusters : ` ~/git/demo2-repo/clusters/bases/rbac/ `

These ClusterRoles exist currently :
```
k get clusterrole | grep gitops- | grep -v weave | awk '{ print $1 }'
gitops-apps-reader
gitops-canaries-reader
gitops-capicluster-reader
gitops-configmaps-reader
gitops-gitopsclusters-reader
gitops-identities-reader
gitops-pipelines-reader
gitops-policies-reader
gitops-reader
gitops-secrets-reader
gitops-templates-reader
```

# TODO
we still need a good rbac definition and tenancy demo

1st step RBAC for admin right for all regular users.
