**works for me (lutz)**

# Policies
The official Policy demo doc lives in Notion. 
https://www.notion.so/Scenario-2-Trusted-Delivery-62eed78e9bdf4664b64d2952d0707255


## Step 1 - Create a policy enabled cluster

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

The kustomization.yaml is used to define all the policy resoucres in the cluster. A **Policy Set** can be used to build subsets of the defined policies and you can use this to define which policies get used for what purpose in our cluster.

The policy-agent values.yaml above refere to the **admission-policy-set** which will be used for the admission controller. There is no Policy Set in the audit section, thus all known policies will be reported on. As you can see in the **grep** output, only the  ControllerMinimumReplicaCountSuperDevTenant/policy.yaml is enforced as port of the admission-policy-set.

If we look at the details of the ControllerMinimumReplicaCountSuperDevTenant/policy.yaml we will find that it will only be enforced on 
```
targets: {kinds: [Deployment], namespaces: [superdevs]}
```
Deployments in the *superdevs* namespace.

## Step 2 - Deploy a violating service

After activating the policy is as part of step 1, we are now deploying a violating service. We do have a simple application that has it's config in the apps/violating-service directory of the management cluster. We can use the Web UI to creathe the PR that will add the kustomization to add this app to our leaf cluster.

Create the superdevs namespace in your cluster. ( This could be part of the app, check if anyone is still using this service. ). 
```
$ ku 10 create ns superdevs
```

* Now fill in the values to add the application : 
![Screenshot from 2022-10-05 15-46-48](https://user-images.githubusercontent.com/2788194/194076375-520730c0-6971-446b-8490-bd870a10d26d.png)

* Create and merge the PR.
Go to the application page of your cluster, you can hit the sync button on **flux-system** kustomization to speed things up.
![Screenshot from 2022-10-06 08-48-03](https://user-images.githubusercontent.com/2788194/194233863-efeaa58d-6c44-40e3-9500-347b0e967f3d.png)

* You can no find the application on the cluster, notice the error : 
![Screenshot from 2022-10-06 08-48-53](https://user-images.githubusercontent.com/2788194/194233808-f8cd77da-ff59-4a27-a1b6-357d40c8cb84.png)

* You can go to the "violation log" tab and see details of the violation ( bug - currently empty v0.9.5 ):
![Screenshot from 2022-10-06 08-49-55](https://user-images.githubusercontent.com/2788194/194233743-fe0c78e4-d4bb-4278-822a-3026635841a2.png)

* You can also find the violation in the global violations log : 
![Screenshot from 2022-10-06 08-50-53](https://user-images.githubusercontent.com/2788194/194233719-d63591ee-8ccd-4ccc-b7d3-1c1980b7e800.png)

## Audit Log Demos

We have 2 different ways to visualize the collected audit information. 

On Prem : Through EFK. [A demo report can be found here](https://kibana.aws.dev.policy.weave.works/goto/b5153050-1c8b-11ed-af48-419b3139feed), please note, it takes 2 min to load this page.

SaaS : We have the managment clusters connected to our Policy SaaS backend. This has nice looking reports. [https://policy.weave.works/home] you will need to be added to the demo team for access. Create an account and contact lutz@weave.works.


