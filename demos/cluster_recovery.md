# How to demo cluster recovery

## Prerequisites

You must already have a deployed cluster on demo3.
The cluster is called: 

* [wordpress-mttr-demo-50-43](https://demo3.weavegitops.com/cluster/details?clusterName=wordpress-mttr-demo-50-43)

The template for creation of the cluster is:

* [lm-edge-nfs-wordpress](https://demo3.weavegitops.com/templates/lm-edge-nfs-wordpress/create)

Deploy the cluster using the default values in this template.

## Running the Demo

When the cluster is running go to the wordpress blog URL:

https://wordpress.weavegitops.com/

You can see a worpdress blog site.

You can log in to the admin page here:

https://wordpress.weavegitops.com/wp-login.php

The admin username and password are saved in onepassword in the WGE SA Demos shared Vault as well as here:

* User: admin
* Password: cj900dsm0cjkeOII

Use the web interface to create a new page in wordpress

https://wordpress.weavegitops.com/wp-admin/post-new.php

Click Publish at the top left to complete the post.


Go back to the main website to show the newly created page.
The next step is to destroy the cluster.

Use the delete cluster feature on the clusters list page:

* https://demo3.weavegitops.com/clusters

Then delete the cluster:
* wordpress-mttr-demo-50-43

![Delete Cluster](https://user-images.githubusercontent.com/1316183/216016268-f3d50bc2-0dee-4661-a9f2-e6c837c2006c.png)

Approve the PR created.

While the cluster is destroyed, explain some of the features of the WGE UI.
Verify the cluster was destroyed by reloading the blog URL and it fails to load a page.

Now go to the git repository and find the latest commit, revert the commit from the merged PR:


![Github Revert](https://user-images.githubusercontent.com/1316183/216016707-89b1563f-f510-4c48-ba49-989753358c67.png)


Watch as the cluster is restored.

Now go to the blog URL and you will see the blog entirely restored with your new post.
