# sa-demos
This repository is dealing with the Weave GitOps demos and demo environments. Please use it to file and update issues that you are seeing. You can also put your demo guides and demo scripts here.

# Demo Status

## Demo3 is currently the stable environment
https://demo3.weavegitops.com
- running 0.9.3 
- use lm-edge-with-lb to provisiong clusters
-- install with cert-manager and weave-policy 0.4.0, set values accountId, clusterID and AGENT_ENABLE_ADMISSION: "0"
- EKS provisioning is broken

## Demo2 is currently unstable
https://demo2.weavegitops.com
- running v0.9.4-rc1
- broken Sources for fleaf cluster listings
- broken LM provisioning
- broken EKS provisioning
