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
![edit](https://user-images.githubusercontent.com/1316183/228941913-3cb597b0-8dad-4e6f-b9f7-b6ec871d80dc.png)

* Show the list of components that are installed

* Click on some of the Values of the Profiles to show they are editable
* Show the secret-store-config Values.yaml
![WGE-wordpress-secret-store-config](https://user-images.githubusercontent.com/1316183/228942217-f6669359-2a0b-4e93-9cb4-74cbf4ac847b.png)

* Explain that the cluster build process pushes a special secret into the cluster at install time 
* This is then used to pull secrets from an external secrets store and sync them into the kubernetes cluster
* Show the secrets in the secrets page

![secrets-list](https://user-images.githubusercontent.com/1316183/228942657-b5281cf1-0521-4b9d-9453-c988f508e8c9.png)


* Now go back to the cluster list, then click on the Wordpress-mttr-demo.. link to show the cluster status

* Now click on Go To Applications
* Show the applications running on the cluster and their status
![wge-wordpress-apps](https://user-images.githubusercontent.com/1316183/228943184-63558f92-4a9d-46a4-a104-8e532400541f.png)

* Show the wordpress application
![wge-wordpress-app-url](https://user-images.githubusercontent.com/1316183/228955645-d2e6fc81-f815-4d1d-b397-b6944e24de15.png)


## Deleting the cluster

* We have received confirmation that the new system has taken over and we can retire this cluster now.
* So, we go ahead and delete it.
* Go to the Cluster list, tick the box next to the Wordpress-mttr-demo.. cluster and click on CREATE A PR TO DELETE CLUSTERS
![wge-create-pr](https://user-images.githubusercontent.com/1316183/228955888-d99bd04c-94b0-403a-9be9-c4a7786ea50b.png)

* Auth to Github
* Create a PR, click on the link to go to github.
* Approve the PR to delete the cluster
![github-merge-pr](https://user-images.githubusercontent.com/1316183/228956163-39d8c66c-7e24-4964-b41b-0ce8b0f5293e.png)

Now we have approved the PR we have complete traceability of who carried out the deletion of the cluster, when and who approved the PR to deploy the change to our production infrastructure.

* Go back to the WGE clusters list
* Show the cluster deleting
![wge-capi-deletion](https://user-images.githubusercontent.com/1316183/228956383-13dfa104-2e76-45b9-8499-7bee519aafdf.png)

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
![github-select-pr](https://user-images.githubusercontent.com/1316183/228956716-55756781-0b16-45f2-8b16-8ce323e3de57.png)

* Now click the Revert button to create a new PR to revert the change
![github-revert-pr](https://user-images.githubusercontent.com/1316183/228956748-7dbc4472-1419-4b99-ab35-edcf813309f5.png)

* Approve this new PR

## Time to Recovery

* Now we have approved the PR, start a timer
* Go back to the WGE clusters list
* Watch the cluster deployment start
![wordpress-cluster-build](https://user-images.githubusercontent.com/1316183/228963816-7da0b920-0d50-4770-9906-ebd96f70a123.png)


* Click on the cluster to show the status, wait for 3 green lights
![wge-cluster-status](https://user-images.githubusercontent.com/1316183/228963938-3c50b065-74e6-464d-bad9-886a37313a9b.png)


* Click on Go To Applications - this will bring up a BLANK page
![wge-apps-blank](https://user-images.githubusercontent.com/1316183/228964057-cf13f102-084a-45a7-a733-6118b769e3f4.png)


* Wait for another 2-3 minutes
* Applications should appear (if they don't hit refresh a few times while waiting)
![wge-apps-reconciling](https://user-images.githubusercontent.com/1316183/228964379-4acab547-c53a-4007-81cf-63809a02bad5.png)


* Then you can see the progress of the restore
* When the wordpress application finishes reconciling
* Click on the wordpress helm chart and show the status.
* Click on the App URL link to show the blog is up and running again with all it's data


* Stop the timer, example total time: 7 minutes 30 seconds

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
