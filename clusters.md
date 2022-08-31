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
Demo1 - 172.16.10
Demo2 - 172.16.20
Demo3 - 172.16.30

You will need to identify a free Cluster number between 6 and 49. Look through the cluster list 

Use the lm-edge template and fill in these values :


![Screenshot from 2022-08-31 10-04-20](https://user-images.githubusercontent.com/2788194/187626487-7424f4d5-8f7d-406b-a7de-ca5058901e3f.png)


