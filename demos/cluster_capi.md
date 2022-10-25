Proven to work on v0.9.5

# How to demo cluster creation

## Prerequisites

1. Install and Connect to tailscale
- Login to [tailscale.com](https://www.tailscale.com) using your Weaveworks Login
- Download tailscale
- Run tailscale with : 
``` 
# tailscale up --accept-routes
```

2. Check connectivity
Ping a machine on one of the Equinix Bare metal environments 
```
$ ping 172.16.20.2
PING 172.16.20.2 (172.16.20.2) 56(84) bytes of data.
64 bytes from 172.16.20.2: icmp_seq=1 ttl=64 time=45.2 ms
```

## Liquid Metal Cluster creation

You will need a **free** cluster number to use for the Name / IP of the cluster. Browse the list of connected clusters and see what might be free. This number needs to be between 5 and 50. 5 and 50 should not be used.

Cluster IP Base : 
* Demo1 - 172.16.10
* Demo2 - 172.16.20
* Demo3 - 172.16.30

Use the **lm-edge** template and fill in these values :

1. Give your Cluster a name that includes the **free** cluster number e.g. "lutz22".
2. We only have cillium as CNI for now
3. We only allows for 1 Control plane machine as this is an edge scenario and we want light wheight clusters. This is configured in the CAPI template.  
4. Re-use that number for the CONTROL_PLANE_VIP
5. Select a bare metal host to install this K8s cluster on (edge use case). It is possibe look at the bare metal hosts on the CLI. Ask lutz if you want to show this.
6. All available K8s versions are known to work
7. Use the Cluster Base and The cluster number add 200 to calulate the LOADBALANCER_IP
8. Deploy at least 2 worker nodes.

![Screenshot from 2022-08-31 11-12-45](https://user-images.githubusercontent.com/2788194/187643487-48a35c5f-07d1-4573-80eb-841f102aa5a9.png)

Scroll down and fill in the Profiles section.

Select prometheus, if you want it and ~~fill in the namespace names~~. The values.yaml and metallb and weave-policy-agent are prefilled and use the above input. Version are currently pinned as part of the [CAPI Template](https://github.com/weavegitops/demo3-repo/blob/95845302b5385de393a97257b0b2aa2be4375f1a/weave-gitops-platform/capi-templates/capmvm-edge.yaml#L7).

![Screenshot from 2022-08-31 11-12-34](https://user-images.githubusercontent.com/2788194/187643480-2230c997-3f27-4960-9d8f-92b092705f5f.png)

## CREATE PULL REQUEST
* Go through the manual auth flow and create the pull request anyway
* Merge the pull request - click the PR link in the pop up box
* You can start a clock for some fun. 
* Download the kubeconfig when the cluster shows in the UI
* The control plane should be reachable within 2-3 min 
* Nodes show up 3-4 min after merge

Helper script : ~/git/cx-presales/lutzdemo/scripts/helper/ku
You can place that in your ~/bin and make it executable. It uses your ~/Downloads dir and the newest kubeconfig with $NUM.

```
$ kubectl  --kubeconfig=~/Downloads/mycluster24.kubeconfig get nodes 
```
or
```
$ ku 24 get nodes
```
if you are too quick or tailscale is not up, connections will fail

* first response you might get is an empty ressource list
* the control plane will show up as not ready initially
* soon after that the worker nodes will join

I’m switching to looking at the pods at this stage
```
$ watch -n 1 ku 24 get pods -o wide -A
```
this will list all the pods as they are getting started
* note that cillium starts up
* note that coredns pods are restarted
```
$ watch -n 1 ku 24 get pods -o wide -A
```

All pods should have working IPs and will see the profiles deploy in layers 
* layer-0: cert-manager & metalb
* layer-1: ingress-nginx and weave-policy-agent
* layer-2: prometheus

## Demo Grafana

Go to the cluster list and click the **grafana** link. This opens the grafana page if tailscale is installed and up and running with --accept-routes.

* user : admin
* pw : prom-operator

## Cluster Updates - Edit cluster

With v0.9.4 we get the capability of editing existing CAPI based clusters to change parameters. This requires that the Template file has the GitOpsCluster resource as the 1st object in the list of objects. The inital provisioning of your cluster will annotate the 1st yaml object of the template file with the values that were used. 

You can find the edit button available in the cluster listt, but only for the clusters where the GitOpsCluster object has been annotated.
![Screenshot from 2022-09-15 13-59-42](https://user-images.githubusercontent.com/2788194/190577634-5416428c-1b4a-40b5-b45c-6fa9d3f1ed5b.png)

Changes that should work change : 
- profiles installed, versions, values.yaml 
- K8s worker nodes ( works for LM, EKS 1-8 nodes ) 

Changes that should/might not work :
- IP address changes
- K8s version changes ( needs more involved procedures )


## Demo Policy deployment

**_Note_** this was tested with weave-policy 0.4.0

We just installed a cluster with the weave-poliy-agent 0.4.0. This version installs only the policy CRDs and no actual policy. This is why you can't find the new cluster in the policy page. You can deploy policy easily with the new add application flow. This essestially adds a kustomization only. The GitRepository or Source needs to exist in the cluster for you to use this flow. 

We do have existing policy in yaml format in our management repository. This makes it easy to deploy this policy to the new leaf cluster.

* Go to Application and fill in the values :

1. Name the Kustomization : policy-cluster$NUM
2. Kustomization Namespace use : policy-system (note the field is titled wrong in 0.9.3) 
3. Select your cluster
4. Select the source : **flux-system**
5. Input the path the the policy definitions in the management repo : weave-gitops-platform/demo-policies

Screenshot for v0.9.3
![Screenshot from 2022-08-31 12-02-18](https://user-images.githubusercontent.com/2788194/187653330-b39a176d-39cd-470d-8a7b-d2e836b72d26.png)

Screenshot for v0.9.4-rc1
![Screenshot from 2022-08-31 12-11-28](https://user-images.githubusercontent.com/2788194/187655601-33bdec72-d6c0-459b-9847-19a72f6d8311.png)

6. Authenticate & Merge the PR

You can now go to Applications find the flux-system kustomization of your cluster and hit the sync button. Or wait a while for reconcilliation. 

![Screenshot from 2022-08-31 12-12-37](https://user-images.githubusercontent.com/2788194/187655697-26997638-a005-405f-a148-3faec5fd4644.png)

You can now find the installed policies in the Policy Tab : 
 
![Screenshot from 2022-08-31 12-13-05](https://user-images.githubusercontent.com/2788194/187655770-e413186e-a67f-4ddc-b68e-2fe7c5c8b038.png)

**Please Note** Policy enforcement is disabled by default in the leaf custers. We are running in audit only mode.

# Optional Cluster Lifecycle Management with CAPI : 
This is GitOps, we can do cluster management through Git.
Use the UI and browse to weave-gitops/apps/capi edit mycluster24.yaml
find the MachineDeployment object
increase the replica count by 3 ( decreasing does not currently work well )
Look at the nodes and the new pods coming up.

# Optional Cluster Demo - Upgrade to a new Kubernetes version. 
* Just follow the Cluster API book. 
* There is a [recording](https://drive.google.com/file/d/1KpP216bEcef5Fh8KNXIoad1TTsSeaOLo/view?usp=drive_web)

