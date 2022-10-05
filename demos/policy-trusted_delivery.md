**work in progress**

# Policies
The official Policy demo doc lives in Notion. 
https://www.notion.so/Scenario-2-Trusted-Delivery-62eed78e9bdf4664b64d2952d0707255

If you want to demo policies in action, please use the following agent-configuration when setting up your policy-demo cluster :
- The following configuration is specific to policy-agent v0.5.0
- You might want to adjust the **accountID** and **clusterID** as the clusterID will show up in the audit reports
- This config enables a aduit.json sink and a admission sink to k8sEvents.
- This config referes to the admission-policy-set that is defined as part of the management repository : ./weave-gitops-platform/demo-policy-library
- There will be a git-source option that can be used in the policy-source section (can be used instead of path, secrect and url)
```
policy-agent:
  image: magalixcorp/policy-agent
  failurePolicy: Ignore

  useCertManager: true
  certificate: ""
  key: ""
  caCertificate: ""

  persistence:
    enabled: false

  config:
    accountId: my-account
    clusterId: my-cluster
    audit:
      enabled: true
      sinks:
        filesystemSink:
          fileName: audit.json
    admission:
      enabled: true
      policySet: admission-policy-set
      sinks:
        k8sEventsSink:
          enabled: true

policySource:
  branch: main
  enabled: true
  path: ./weave-gitops-platform/demo-policy-library
  secretRef: flux-system
  url: ssh://git@github.com/weavegitops/demo2-repo
```

The above configuration refences the base kustomization.yaml : 
[demo2-repo/weave-gitops-platform/demo-policy-library/kustomization.yaml](https://github.com/weavegitops/demo2-repo/blob/main/weave-gitops-platform/demo-policy-library/kustomization.yaml)
and the admission controller will use the **admission-policy-set** :
```
apiVersion: pac.weave.works/v2beta1
kind: PolicySet
metadata:
  name: admission-policy-set
spec:
  id: admission-policy-set
  name: admission-policy-set
  filters:
    tags: [tenancy]
    # ids:
    #   - weave.policies.containers-minimum-replica-count
    # - weave.policies.tenancy.containers-minimum-replica-count
```
That is any policy that has a **tenancy tag** set. You could filter by catergories or standards as well. If you are referring to **ids** directly, the will override the dynamic filters.

This is currently only set for these two policies : 
```
$ cd ~/git/demo2-repo/weave-gitops-platform/demo-policy-library
$ grep -irl tenancy | grep policy.yaml
./policies/ControllerMinimumReplicaCountSuperDevTenant/policy.yaml
./policies/HPAMinimumReplicaCount/policy.yaml
```

The kustomization.yaml is used to define the resoucres in the cluster. The policy-agent values.yaml setting will chose how these resources are treated. Are the reported on only / used for audit? Or will they be enforced through the Admission controller. The Policy-Set object **admission-policy-set** conrtols which policies will be used for the admission controller. In the configuration above, only ControllerMinimumReplicaCountSuperDevTenant/policy.yaml is enforced.

If we look at the details of the ControllerMinimumReplicaCountSuperDevTenant/policy.yaml we will find that it will only be enforced on 
```
targets: {kinds: [Deployment], namespaces: [superdevs]}
```
Deployments in the *superdevs* namespace.
