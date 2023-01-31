# How to demo cluster recovery

## Prerequisites

You must already have a deployed cluster.
The cluster is called: 

* wordpress-mttr-demo-50-43

The template for creation of the cluster is:

* lm-edge-nfs-wordpress

Deploy the cluster using the default values in this template.

## Running the Demo

When the cluster is running go to the wordpress blog URL:

https://wordpress.weavegitops.com/

You can see a worpdress blog site.

You can log in to the admin page here:

https://wordpress.weavegitops.com/wp-login.php

The admin username and password are:

* Username: admin
* Password: cj900dsm0cjkeOII

Use the web interface to create a new page
Go back to the main website to show the newly created page.
The next step is to destroy the cluster.

Use the delete cluster function in the UI to delete the cluster:

* wordpress-mttr-demo-50-43

Approve the PR created.

While the cluster is destroyed, explain some of the features of the WGE UI.
Verify the cluster was destroyed by reloading the blog URL and it fails to load a page.

Now go to the git repository and find the latest commit, revert the commit in a new PR.
Watch as the cluster is restored.

Now go to the blog URL and you will see the blog entirely restored with your new post.

