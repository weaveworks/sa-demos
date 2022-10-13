# Progressive Delivery Demo

see also [Progressive Delivery in Notion](https://www.notion.so/weaveworks/Progressive-Delivery-demo-script-004618e154d04a8b8beed506a7d39174) 

## Overview

Demonstrate how an application team would deploy an update to an application in a controlled progressive manner utilizing Flagger from within the Weave GitOps UI. Feature the observability functionality built into the “Delivery” section of WGE. Be able to show a failed roll-out as well as automated rollback while maintaining application resiliency.

This is not a demonstration of the advanced features and functionality of Flagger, it is designed as a simple demonstration of how an application team would use WGE to roll out a new version.

## Prerequisites

*   [demo3](https://demo3.weavegitops.com/applications/delivery) environment with Delivery section enabled.
*   [progress-delivery-demo32](https://demo3.weavegitops.com/cluster/details?clusterName=progress-delivery-demo32) cluster with flagger, load generator, and nginx installed.
*   [podinfo application](http://progress-delivery-demo32.weavegitops.com/) running and accessible.
*   [canary.yaml](https://github.com/weavegitops/demo3-repo/blob/main/apps/podinfo/templates/canary.yaml) defined.
*   Access to demo3 [git repo](https://github.com/weavegitops/demo3-repo/tree/main/clusters/default/progress-delivery-demo32/apps) to update app deployment.

## Presentation Slides

The presentation slides for this demo are available in gdrive here:

*   [https://docs.google.com/presentation/d/1NLFh1V3tk7mRrvbD55BmleUsHOpSGbAv-4MDUElez0U](https://docs.google.com/presentation/d/1NLFh1V3tk7mRrvbD55BmleUsHOpSGbAv-4MDUElez0U)

These include diagrams of the process and a summary of the technologies used.

## Demo Flow

After introducing the concept of canary deployments and the importance of Flagger as an automation tool, we first demonstrate a failed deployment and prove that the production application is unaffected. We then demonstrate a successful application upgrade in a controlled manner and prove the upgrade becomes available to the end users seamlessly.

1.  Show current status of [progress-deliver-demo32 cluster](https://demo3.weavegitops.com/cluster/details?clusterName=progress-delivery-demo32)
2.  Drill into [Applications](https://demo3.weavegitops.com/applications?filters=clusterName%3A%20default%2Fprogress-delivery-demo32) for that cluster
3.  Show flagger and ingress-nginx are installed and running (optionally drill into flagger to show loadbalancer and Prometheus components installed)
4.  Show status of p[odinfo application](https://demo3.weavegitops.com/helm_release/details?clusterName=default%2Fprogress-delivery-demo32&name=podinfo&namespace=test).  Note current version number.
5.  Show current running [application UI](http://progress-delivery-demo32.weavegitops.com/).  Note current version number matches previous step.
6.  Show pre-configured [canary.yaml definition](https://github.com/weavegitops/demo3-repo/blob/main/apps/podinfo/templates/canary.yaml) in repo.  Point out Flagger variables (60 seconds, analysis every 10 seconds, max 10 failures, 5% weighting step until 50%, >99% success rate of curl test for token).  Spend as little or as much time here as your audience understands or is familiar with flagger.
7.  Show current [Delivery page](https://demo3.weavegitops.com/applications/delivery/4418f6f8-a88a-4d04-a50d-f556ccbce6e0/details?clusterName=default%2Fprogress-delivery-demo32&name=podinfo&namespace=test) to show success of previous upgrade and where we will monitor our upcoming upgrades.  Show Primary replicas are 1 and everything else is 0.
8.  Commence upgrade by editing [podinfo.yaml](https://github.com/weavegitops/demo3-repo/blob/main/clusters/default/progress-delivery-demo32/apps/podinfo.yaml) to point to “unstable” release - image: "ghcr.io/enekofb/podinfo:error"
9.  Monitor [Delivery page](https://demo3.weavegitops.com/applications/delivery/4418f6f8-a88a-4d04-a50d-f556ccbce6e0/details?clusterName=default%2Fprogress-delivery-demo32&name=podinfo&namespace=test) as upgrade is analyzed.  Show Deployment image version changes, but not Primary yet.  Note increasing failure count every 10 seconds.  Show Events and Analysis tab.
10.  Flip over to the application UI and note the version number stays the same and the application is accessible as analysis is taking place.
11.  Return to Delivery page and wait for failure.
12.  After complete failure, return to application UI and point out that flagger has protected your running production application despite bad code being introduced.  Discuss how GitOps allows you to see who introduced that changed, who approved it, and easily allows you to revert to the previous known good configuration.
13.  Simulate releasing a corrected version upgrade by again editing the [podinfo.yaml](https://github.com/weavegitops/demo3-repo/blob/main/clusters/default/progress-delivery-demo32/apps/podinfo.yaml) this time to point to a stable release - image: "ghcr.io/stefanprodan/podinfo:6.0.4" (note: version must be different than initial setting)
14.  Return to Delivery page, show Events tab as analysis starts, show Objects tab as canary replicas are ramped up at new version.
15.  Flip over to application UI to show it is maintaining the current version until analysis is complete.
16.  Continue to monitor Delivery page noting no increasing errors and no events.
17.  Upon nearing completion (50%), flip to Objects tab and show that the primary becomes the new version and the canary is spun down.
18.  Return to application UI to show it is now running the upgraded version.
19.  Return to slides to recap what happened and the value of protecting your production applications. 

## Reset and cleanup

*   If possible, return podinfo.yaml to original version or decrement version.

## Demo recording

You can watch the internal [recording here](https://drive.google.com/file/d/1-rhaTCSi-dtQnPqRwofPIhLU7wwVSmav/view?usp=sharing)
