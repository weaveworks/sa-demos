# This is not tested, we are working on this!

# External-Secrets

**Secrets should not be stored in Git**

We can use a centralised secret manager like Hashicorp Vault. Or AWS Secret Manager. Since we are using EKS for the management clusters any way, we 
are using AWS Secret Manager.

(External Secrets Operator)[https://external-secrets.io/] is an that will connect to centralized secret stores to read secrets and create these secrets in cluster.

There is essentially 2 scenarios for external-secrets manager

## A) Manage Secrets in the Management Cluster

Access to AWS Secrets Manager can be allowed by a Secret or by IRSA

Question: Which permissions do I need on AWS to allow access to AWS Secrets Manager?
See : (external secrets provider documentation)[https://external-secrets.io/v0.5.9/provider-aws-secrets-manager/]

Create an IAM Role with the following permissions, this would allow access to all demo2-* keys :

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetResourcePolicy",
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret",
        "secretsmanager:ListSecretVersionIds"
      ],
      "Resource": [
        "arn:aws:secretsmanager:eu-central-1:482649550366:secret:demo2-*"
      ]
    }
  ]
}
```

Create a SecretStore to configure External Secrets to reach out to AWS


The following does not use IRSA, does it?

```
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: secretstore-sample
spec:
  provider:
    aws:
      service: SecretsManager
      # define a specific role to limit access
      # to certain secrets.
      # role is a optional field that 
      # can be omitted for test purposes
      role: iam-role
      region: eu-central-1
      auth:
        secretRef:
          accessKeyIDSecretRef:
            name: awssm-secret
            key: access-key
          secretAccessKeySecretRef:
            name: awssm-secret
            key: secret-access-key
```


## B) Manage Secrets in the Leaf Clusters

Question : Does it make a difference if the leaf cluster is on AWS or not?

