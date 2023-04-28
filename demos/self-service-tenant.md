# Prepare the demo

We need:
* A browser profile in Chrome with the google identity: demo-limited-user@weave.works
* The Onepassword entry for the SA Demo Limited User in the Weaveworks Shared Vault

For Self Service Cluster (optional):
* Check that cluster: **devteam1-hostxx-cluster30** is not deployed
* If it is delete the cluster first
* Make sure you have access to the fleet cluster repository: **https://github.com/weavegitops/demo3-repo**
* Check that you can raise a Pull Request against this repository

For Self Service Application:
* Check that the cluster: **devteam1-hostxx-cluster30** is already deployed as this will be the target cluster
* Check you have access to the application developers repository: **https://github.com/weavegitops/application-promotion-podinfo**
* Check that you can make a Pull Request against this repository
* Install the following extensions in VSCode:
    * Gitops Tools for Flux
        * Go to the Extension settings and turn on Templates
    * Github Pull Requests and Issues

The tenant has already been prepared in the demo environments.

RBAC and policies and namespaces have been created and allocated.

Templates have been created for the tenant's use.

# Self Service Tenancy demo guide

The platform team is responsible for this preparation of a tenant environment.

The tenant in this demo has been set up with access to deploy their own clusters from a template.

The tenant also has access to a couple of shared clusters as well, but only to their allocated namespaces.

The platform team can see this configuration in the Workspaces tab of the WGE UI.

The tenant can then log in to WGE UI with their own identity.

# Self Service Deploying a Cluster (optional)

To deploy a cluster:
* First check to see if the cluster: **devteam1-hostxx-cluster30** is already deployed.
* Delete the cluster first
* Go to Create Cluster and use the template: **lm-edge-devteam1-self-service**
* Leave all values as the defaults
* Create the PR and merge it
* Go to the clusters page and watch as the cluster is added
* Show the cluster details page and the 3 green ticks as the cluster deploys
* Download the new kubeconfig, use some CLI commands to add it to your local kubeconfig
* Show the cluster applications list and wait for the applications to come up

# Self Service Deploying an Application

To deploy a Helm chart application with a pipeline:
* First check to see if the cluster: **devteam1-hostxx-cluster30** is already deployed.
* Now go to Templates
* Use the template: **app-podinfo-chart-and-pipeline-devteam1**
* Leave the default values
* Create the PR and merge it
* This will deploy the application to the target cluster in the separate namespaces and create the pipeline
* Show the pipeline in the Pipelines view
* Show the Delivery view and that each deployment has initialised

# Making a code change to the application in VSCode

Make a code change to the application:
* Make a new branch for the repo: **application-promotion-podinfo** using the Source Control extension
* Use the explorer view to find and edit the following files:
  * Edit the file: **cmd/podinfo/main.go** to change the HTML colour on line: 43
  * Edit the file: **pkg/version/version.go** to increment the version number
  * Edit the file: **charts/podinfo/Chart.yaml** to increment BOTH the app version and helm chart version
  * Edit the file: **charts/podinfo/values.yaml** to increment the container version in the values
* Stage all the changes in the Source Control section
* Make a commit stating: "release container X.X.X and Helm chart X.X.X"
* Publish the branch, which should also prompt you to create a Pull Request
* Go ahead and create the pull request - which VSCode will show you a summary

# Ephemeral Environments

Now we have an open pull request this will automatically deploy the application from the Pull Request to a test environment where we can see our changes deployed before we merge the pull request.

To see the application:
* Go to GitopsSets in the WGE UI
* Click on the app-promo-podinfo-pullrequests Gitops Set
* There should be 2 resources created, click on the HelmRelease
* Show the Helmrelease and see the app-url, click on it to show the application

# Merging and Releasing

Now we can release the Helm chart with the new container version.

* Go back to VSCode to view the Pull Request summary
* Click on the Merge button to merge the code and run the Helm release process
* Go back to WGE and show the pipelines view and explain that once the chart is released it will deploy to dev automatically
* Show the Delivery view and wait for the dev podinfo starts to rollout the application
* Show the application URL to show the colour change happening, links are here:
* http://demo3-cluster30.weavegitops.com/dev/podinfo/
* http://demo3-cluster30.weavegitops.com/stg/podinfo/
* http://demo3-cluster30.weavegitops.com/prd/podinfo/
* The canary analysis will be completed successfully

# Promoting the Application

Now we want to promote the application across environments.

* Go back to the Pipelines view to show the updated dev environment version
* Show the approve button for promoting to the stg environment, click the button
* A PR will be created, click on the Link to the PR to go to github
* Approve the PR to promote the version to stg
* Go back to WGE and go to the Delivery view to see the stg application rollout
* Once completed, go back to the Pipelines view
* Approve the promotion to prd and click on the PR link, approve the PR in github
* Go back to WGE and go to the Delivery view to see the prd application rollout
* Once completed go back to the Pipelines view to show the finished pipeline

# Demo completed or repeat

* The demo is now completed and is ready to run again from Making a Code Change to rollout the next version

# Demo Reset

## Application Self Service

To reset the demo we need to remove the pipeline and Helmrelease objects these are located in the demo3 repo:
* clusters/management/pipelines/app-podinfo-chart-and-pipeline-devteam1.yaml
* clusters/devteam1/devteam1-host25-cluster30/app-podinfo-dev
* clusters/devteam1/devteam1-host25-cluster30/app-podinfo-stg
* clusters/devteam1/devteam1-host25-cluster30/app-podinfo-prd

## Cluster Self Service

To reset the cluster deployment, just delete the cluster:
* Use the Clusters view in the WGE UI
* Select the cluster **devteam1-host25-cluster30**
* Create a PR to Delete the Cluster
* Approve the PR to carry out the deletion
