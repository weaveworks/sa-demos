# Documenting the state of tf-controller 

There are multiple ways to install tf-controller. We just went with the manual tf-controller installation as helm chart.

```
# Add tf-controller helm repository
helm repo add tf-controller https://weaveworks.github.io/tf-controller/

# Install tf-controller
helm upgrade -i tf-controller tf-controller/tf-controller \
    --namespace flux-system
```

As an alternative, we could create a Helmrelease and a Helmchart yaml and thus automate the setup with GitOps.
