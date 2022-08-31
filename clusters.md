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

## Liquid Metal Cluster creation

You will need a free cluster Number to use for the Name / IP of the cluster. 

Cluster IP Base : 
* Demo1 - 172.16.10
* Demo2 - 172.16.20
* Demo3 - 172.16.30

You will need to identify a free Cluster number between 6 and 49. Look through the cluster list 

Use the lm-edge template and fill in these values :

![Screenshot from 2022-08-31 11-12-45](https://user-images.githubusercontent.com/2788194/187643487-48a35c5f-07d1-4573-80eb-841f102aa5a9.png)

Select prometheus, if you want it and fill in the namespace names. The values.yaml and metallb and weave-policy-agent are prefilled and use the above input. Version are currently pinned as part of the [CAPI Template](https://github.com/weavegitops/demo3-repo/blob/95845302b5385de393a97257b0b2aa2be4375f1a/weave-gitops-platform/capi-templates/capmvm-edge.yaml#L7).

![Screenshot from 2022-08-31 11-12-34](https://user-images.githubusercontent.com/2788194/187643480-2230c997-3f27-4960-9d8f-92b092705f5f.png)



