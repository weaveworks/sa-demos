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




