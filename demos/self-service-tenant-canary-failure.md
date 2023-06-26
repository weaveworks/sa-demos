# Making a code change to the application that fails progressive delivery

# Making a code change to the application in VSCode (failure)

Make a code change to the application:
* Make a new branch for the repo: **application-promotion-podinfo** using the Source Control extension
* Use the explorer view to find and edit the following files:
  * Edit the file: **cmd/podinfo/main.go** to change the HTML colour on line: 43
  * Edit the file: **pkg/version/version.go** to increment the version number
* Stage all the changes in the Source Control section
* Make a commit stating: "change the background colour"
* Make another change to increment the version of the Helm chart
  * Edit the file: **charts/podinfo/Chart.yaml** to increment BOTH the app version and helm chart version
  * Edit the file: **charts/podinfo/values.yaml** to modify the nodePort and externalPort to 9696 instead of 9898
* Stage all the changes in the Source Control section
* Make a commit stating: "Release Helm chart version x.x.x"
* Publish the branch, which should also prompt you to create a Pull Request
* Go ahead and create the pull request - which VSCode will show you a summary

# Ephemeral Environments (looks OK)

Now we have an open pull request this will automatically deploy the application from the Pull Request to a test environment where we can see our changes deployed before we merge the pull request.

To see the application:
* Go to GitopsSets in the WGE UI or VScode plugin
* Click on the app-promo-podinfo-pullrequests Gitops Set
* There should be 2 resources created, click on the HelmRelease
* Show the Helmrelease and see the app-url, click on it to show the application

# Merging and Releasing (Failed release)

Now we can release the Helm chart with the new container version.

* Go back to VSCode to view the Pull Request summary
* Click on the Merge button to merge the code and run the Helm release process
* Go back to WGE and show the pipelines view and explain that once the chart is released it will deploy to dev automatically
* Show the Delivery view and wait for the dev podinfo starts to rollout the application
* The canary analysis will FAIL

# Making a code change to the application in VSCode (fixing the issue)

Make a code change to the application:
* Make a new branch for the repo: **application-promotion-podinfo** using the Source Control extension
* Use the explorer view to find and edit the following files:
* Make a change to increment the version of the Helm chart
  * Edit the file: **charts/podinfo/Chart.yaml** to increment the helm chart version
  * Edit the file: **charts/podinfo/values.yaml** to modify the nodePort and externalPort back to 9898 to fix the issue
* Stage all the changes in the Source Control section
* Make a commit stating: "fix Helm chart and release x.x.x"
* Publish the branch, which should also prompt you to create a Pull Request
* Go ahead and create the pull request - which VSCode will show you a summary


# Merging and Releasing (Fixed release)

Now we can release the Helm chart with the new container version.

* Go back to VSCode to view the Pull Request summary
* Click on the Merge button to merge the code and run the Helm release process
* Go back to WGE and show the pipelines view and explain that once the chart is released it will deploy to dev automatically
* Show the Delivery view and wait for the dev podinfo starts to rollout the application
* The canary analysis will now succeed

We can continue with the promotion worflow as now the application will pass the health check and create a Pull Request
 
