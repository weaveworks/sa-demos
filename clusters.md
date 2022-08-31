# How to demo cluster creation

## Prerequisites

1. Install and Connect to tailscale
- Login to (tailscale.com)[https://www.tailscale.com] using your Weaveworks Login
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

You will need a free cluster Number to use for the Name / IP of the cluster. 

Cluster IP Base : 
* Demo1 - 172.16.10
* Demo2 - 172.16.20
* Demo3 - 172.16.30

You will need to identify a free Cluster number between 6 and 49. Look through the cluster list 

Use the **lm-edge** template and fill in these values :

![Screenshot from 2022-08-31 11-12-45](https://user-images.githubusercontent.com/2788194/187643487-48a35c5f-07d1-4573-80eb-841f102aa5a9.png)

Select prometheus, if you want it and fill in the namespace names. The values.yaml and metallb and weave-policy-agent are prefilled and use the above input. Version are currently pinned as part of the [CAPI Template](https://github.com/weavegitops/demo3-repo/blob/95845302b5385de393a97257b0b2aa2be4375f1a/weave-gitops-platform/capi-templates/capmvm-edge.yaml#L7).

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

# Optional Cluster Lifecycle Management with CAPI : 
This is GitOps, we can do cluster management through Git.
Use the UI and browse to weave-gitops/apps/capi edit mycluster24.yaml
find the MachineDeployment object
increase the replica count by 3 ( decreasing does not currently work well )
Look at the nodes and the new pods coming up.

# Optional Cluster Demo - Upgrade to a new Kubernetes version. 
* Just follow the Cluster API book. 
* There is a [recording](https://drive.google.com/file/d/1KpP216bEcef5Fh8KNXIoad1TTsSeaOLo/view?usp=drive_web)

