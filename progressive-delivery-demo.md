**Document status: initial draft**

## TODOs
- [ ] Define hosts in Route 53
- [ ] Create list of available hosts and reference below
- [ ] Confirm RBAC required for leaf cluster inc. potential PD bug
- [ ] Check OIDC setup
- [ ] Compare with standalone PD demo and demo3 cluster setup

# Overview

This page details everything needed to perform a progressive delivery demo with Weave GitOps Enterprise. 

The demonstration is broken into two distinct sections targeting different persona, with two stories for each.
A demo can focus on one or both personas depending on audience and time available.

# Platform Operator user stories

## 1) As a platform operator, I can configure a leaf cluster to support progressive delivery of applications

1. Choose an available hostname from //TODO add list
1. Create a new cluster following the guide here https://github.com/weaveworks/sa-demos/blob/main/clusters.md,
**additionally** adding the Flagger profile and specifying the namespace as `flagger`. 
Verify that the following *required* components are installed and available:
    1. flagger (via flagger profile)
    1. flagger-loadtester (via flagger profile)
    1. prometheus (via flagger profile)
    1. nginx-ingress (via nginx profile)
    1. metallb (via metalb profile)

## 2) As a platform operator, I can configure appropriate RBAC for application teams to make use of progressive delivery
//TODO expand on the below and check with DavidS
1. Create tentant file
1. Generate RBAC using `gitops` CLI
1. Commit to cluster repository and reconcile change
1. Create source object for app repository
1. Commit to cluster repository and reconcile change

# Application Operator user stories

## 3) As an application operator, I can onboard a new application
1. app repository from Story 2 must contain sample application
1. Login to WGE using OIDC
1. Add new kustomization via add app to leaf cluster using app repository source from Story 2

## 4) An an application operator, I can release a new version of an application via progressive delivery
1. Update deployment manifest in app repository from Story 2 to new version
1. Sync app kustomization to reconcile change to cluster
1. Navigate to Delivery UI to observe rollout
