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



![Screenshot from 2022-08-31 09-45-29](https://user-images.githubusercontent.com/2788194/187624871-f8257a29-f04f-481e-bddf-d07add578e54.png)
