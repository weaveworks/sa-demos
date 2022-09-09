# work in progress

# This documents describes the deployment of OIDC in our demo environments

We need to use Dex to configure OICD Logins for Weave Gitops as Google does not provide group information directly but through a 2nd call. Dex implements getting 
group information from OIDC as well.

We need 3 Secrets for Dex. Long term these should be in AWS Secrets Manager, 1st step is to define them directly on the the CLI and use kubectl to 
install them in the managment cluster. 

The json files for the following secrets are save in a 1P vault. "WGE SA Demos"

- dex-oauth-app-credentials are associated with the definition of the Web App (Dex) in Google.
- dex-google-sa is the service account that allows access to Google objects.
