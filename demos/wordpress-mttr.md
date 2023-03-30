# Wordpress MTTR Demo

This demo is very simple to run

# Preparation

Open Onepassword and find:
**Wordpress Admin Cluster Recovery Demo**

Open the link to the wordpress admin login:
e.g.
https://wordpress.demo3.weavegitops.com/wp-login.php

Use the password to log in as the admin user.

# Demo Flow

* Show the main page of the blog

* Add a post to the blog
![wordpress-add-new-post](https://user-images.githubusercontent.com/1316183/228937272-abd16e0b-f8d3-4d49-b7b7-9b262599fb55.png)

* Publish the new post
![wordpress-publish-post](https://user-images.githubusercontent.com/1316183/228937251-1f40fca1-419b-4f64-a6c8-70c235d4e7a3.png)

* Show the blog and the new post

## Migration of the Blog

We have been instructed to retire the wordpress blog as it has been replaced with an alternative system.

Once confirmation is received that all live services have been moved to the new site, we can retire the cluster.

## Showing the cluster

* Go to the WGE demo environment and log in
* Go to the clusters list

* Find the cluster: **wordpress-mttr-demo...**
![WGE-wordpress-cluster-list](https://user-images.githubusercontent.com/1316183/228941659-69c7abb1-6230-4e87-b32b-a14bfcbd5ef4.png)

* Click on the cluster edit button to view the details of the cluster
* Show the list of components that are installed

* Click on some of the Values of the Profiles to show they are editable
* Show the secret-store-config Values.yaml
* Explain that the cluster build process pushes a special secret into the cluster at install time 
* This is then used to pull secrets from an external secrets store and sync them into the kubernetes cluster

* Now go back to the cluster list, then click on the Wordpress-mttr-demo.. link to show the cluster status

* Now click on Go To Applications
* Show the applications running on the cluster and their status
* Show the wordpress application

## Deleting the cluster

* We have received confirmation that the new system has taken over and we can retire this cluster now.
* So, we go ahead and delete it.
* Go to the Cluster list, tick the box next to the Wordpress-mttr-demo.. cluster and click on CREATE A PR TO DELETE CLUSTERS
* Auth to Github
* Create a PR, click on the link to go to github.
* Approve the PR to delete the cluster (after a review of course)

Now we have approved the PR we have complete traceability of who carried out the deletion of the cluster, when and who approved the PR to deploy the change to our production infrastructure.

* Go back to the WGE clusters list
* Show the cluster deleting
* When the cluster is deleted - should take less than 2 minutes
* Now go back to the wordpress blog URL to verify that the blog has now gone.

## EMERGENCY - INCIDENT RAISED

We are contacted by an operations team that some alarms are going off and some production systems are having an outage.

It turns out one of the production systems **had not been migrated to the new blog server at all!!**

## Initiate IMMEDIATE Recovery

Let's recover the cluster

* From the WGE clusters list click on Open Pull Requests

* Click on Closed Pull Requests
* Click on the Delete Cluster pull request at the top
* Now click the Revert button to create a new PR to revert the change
* Approve this new PR

## Time to Recovery

* Now we have approved the PR, start a timer
* Go back to the WGE clusters list
* Watch the cluster deployment start
* Click on the cluster to show the status, wait for 3 green lights
* Click on Go To Applications - this will bring up a BLANK page
* Wait for another 2-3 minutes
* Applications should appear (if they don't hit refresh a few times while waiting)
* Then you can see the progress of the restore
* When the wordpress application finishes reconciling
* Click on the wordpress helm chart and show the status.
* Click on the App URL link to show the blog is up and running again with all it's data


* Stop the timer

## Explain Gitops Recovery

How does it work?

We use the following:
1. Git to store in version control the entire cluster definition, with all the platform applications and wordpress application configuration
2. An NFS server to store the database data and any image data for the blog
3. An external secret store provided by AWS Secret Manager

When we revert the commit that deletes the cluster we rebuild all the files in the git repository.

This deploys a new cluster with the same parameters, secrets and backend storage as the deleted cluster.

This restores all the applications to their previous state.

# Demo Reset

No demo reset necessary, you are ready to run it again straight away
