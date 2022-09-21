# Application promotion demo

## Demo Features

1. Demostrate GitOps and policy checking across a multi-stage build pipeline.
2. Show a development environment with "instant" image deployment.
3. Show a promotion pipeline to the production environment that requires a pull request to deploy a new image.
4. Show one design pattern for GitOps application promotion: monorepo, instant developer deployment, pull request gate.
5. Use weave-policy-validator to check PRs for policy compliance.

## Environments

There are 5 environments for this demo.  These are leaf clusters that represent each environment.

- `dev` and `dev-test`, for the developers use,
- `uat` or user acceptance testing, used by the testers,
- `stg` or staging, final checks prior to deployment to production,
- `prod` or production.

## Application

The application is a copy of the podinfo application.  
This repo has a copy of podinfo that is customised for this demo.

Versions are part of the demo and so are incremented independently of the source.

The podinfo app source is here:  
- `cmd` - source for the container code
- `pkg` - the metadata for the container build 

The Helm chart source is here:
- `charts/podinfo` - Helm chart to release the podinfo application to environments

## Deployments

The deployment manifests are in the environments directory:
- `environments/dev` - the kubernetes manifests for the dev environment
- `environments/dev-test` - the Helmrelease to deploy the latest version
- `environments/uat` - the Helmrelease to deploy a fixed version
- `environments/stg` - the Helmrelease to deploy a fixed version
- `environments/prod` - the Helmrelease to deploy a fixed version

## Developer Experience

The process for a developer working on the app code is as follows:
1. Developer makes some changes to the app code in the cmd directory
2. Developer pushes the changes to a feature branch, this runs a build of the container with a temporary tag, based on the commit SHA and a timestamp.
3. The `dev` environment deploys the new container image, so the developer can see the new changes.
4.  The developer continues to iterate the code and each push deploys a new image to `dev`
5.  When the developer is ready to release the changes, he needs to set the new version number of the container and update the Helm chart
6. The developer then creates a PR from the feature branch, this starts the CI testing
7. The CI tests will check the versions have been incremented, validate against the policies, build the new container, validate the Helm chart and deploy a test of the Helm chart.
8. If the tests do not pass, the developer can increment changes on the PR until all the tests pass.
9. When all the tests pass, the PR can be merged.  This will release the new version of the Helm chart.
10. The `dev-test` environment will then be updated with the new Helm chart and the new container.
11.  If the Helm chart deploys correctly to the `dev-test` environment and is healthy a PR is automatically created to update the fixed version number for the `uat` environment.
12.  This is then merged, which deploys the new Helm chart and container to the `uat` environment.
13. If the `uat` deployment is healthy, then a PR is created for the `stg` environment.
14. This is merged and the Helm chart is deployed to `stg` and a PR created for `prod`.
15. Finally the PR is merged to deploy the Helm chart to `prod`.

## Developer Visibility

There is feedback on the deployment of each change to the repo from the Kustomize object that syncs the objects to each environment.  These are shown as green ticks against each commit.
However, this does not show the status of the Helm chart deployed, only the Kustomization.  This is useful for the `dev` environment, because that uses Kustomize to deploy the app.  It is much less useful for the rest of the environments that deploy a Helm chart as any error in the Helm chart deployment would not make the Kustomize fail.

TODO: Maybe it is possible to add a Health Check to the Kustomize that would only pass on successfult podinfo deployment that would improve visibility.

Each environment after `dev` will create a PR for the next environment on successful deployment of the Helm chart, so if no PR shows up, the Helm deployment failed.

You can also see the version of the Helm chart deployed to each cluster using the WGE applications UI as shown here:

![Screenshot from 2022-09-21 12-20-23](https://user-images.githubusercontent.com/1316183/191491375-077b6a51-bafa-4a68-801c-625736da0e8e.png)

## Techniques used

The `dev` environment uses Image Automation to watch the container registry for tags and will select the latest tag for deployment.  The Image Automation will then update the tag in the deployment.yaml and automatically merge the change to the repo.

The `dev-test`, `uat`, `stg` and `prod` environments all use the Helm release and an Alert sent to the github repo to trigger the PR for the next environment.  They also use an Alert to notify the github repo when the kustomize successfully syncs from each commit.

## Demo Flow

The demo is to show the developer experience to customers.
Each stage of deployment takes about 2 minutes, but can be as long as 4 minutes if you're unlucky.
This is quickly enough that you can talk about each of the techniques used and the developer feedback in between each deployment step.

### Suggested demo flow

1. Make sure the repo is up to date with the main branch:

```
        git checkout main
        git pull
```

2. Create a feature branch

```
        git checkout -b my-feature
```

3. Make some changes to the container code, for example, change the message, colour or image URL

```
        vi cmd/podinfo/main.go
```

4. Commit and push the code to the repo

```
        git add cmd/podinfo/main.go
        git commit -m 'my change'
        git push
```

5. Now show the podinfo app from `dev` in the UI in one part of your screen while showing the code that changes in the repository from the github UI, i.e. `environments/dev/podinfo/deployment.yaml`
6. Optional - To use the policy check modify the replicaCount to 1 as detailed below.
7. Now as the developer, I want to release my new container with a new Helm chart:
- Increment the version of the container:  

```
            vi pkg/version/version.go
```

- Modify the Helm chart to deploy the new container version using the tag:  

```
            vi charts/podinfo/values.yaml
```

- Increment the version of the Helm chart:  

```
            vi charts/podinfo/Chart.yaml
```

- Now commit and push the changes:  

```
            git add *
            git commit -m 'release the container and helm chart'
            git push --set-upstream origin my-feature
```

- Now go to the github UI and create a PR from the feature branch
7. Show the process of CI testing running by showing the Github Actions:
      -  https://github.com/weavegitops/application-promotion-podinfo/actions
8. Once the process completes, merge the PR to release the Helm chart.
9. While the chart releaser is running from Github Actions also show the podinfo app on the `dev-test` environment in another part of your screen.  
10. When you see the podinfo app change, then show the PR that was created:
      -  https://github.com/weavegitops/application-promotion-podinfo/pulls
11. Approve the PR to `uat` and then show the podinfo app on the `uat` environment in one part of your screen while showing the code changes from the PR in github.
12. When the podinfo app updates, then show the PR for the `stg` environment.
13. Approve the PR and show the `stg` environment podinfo app in one part of the screen while showing the feedback in commits from each environment and the actions that create the PRs.
14. Once the podinfo app updates in `stg`, show the PR for `prod`.
15. Approve the PR for `prod` and show the `prod` environment podinfo app in part of the screen while showing the values that can be set in the Helmrelease object in `environments/prod/podinfo-helmrelease.yaml` that override the default values.yaml in the Helm chart.
16. Once the podinfo updates in `prod` the demo is completed.

## Policy Checks

To demonstrate the policy checks:

1. Make a change the infringes the replicas policy, by changing the number of replicas to 1.
2. This can be done by changing the values in the Helm chart.
3. To modify the Helm chart values, change replicaCount to 1:

```
        vi charts/podinfo/values.yaml
        git add charts/podinfo/values.yaml
        git commit -m 'reduce replicas to 1 for the Helm chart'
        git push
```

To run the policy check create a PR for the feature branch.
The policy check will create a fix for the violation automatically, just approve the PR for the fix to then re-run the policy check so it succeeds.

## Tidying up

If the demo runs to completion, then no tidy up is needed.

If however the demo did not run to completion, then you need to approve each PR into productio
n.

Also, if the changes you made are ugly like a really bad colour, then please reset the colour
to the default and promote the changes across all the environments to reset the demo.

## Troubleshooting

The github actions are all detailed in `.github/workflows/`
Look there for any code that may need updating for the github actions if there is a problem.

The deployment to each demo environment is depedent on a cluster being set up in advance that points a Kustomization to sync the manifests in the `environments/` directory for each cluster.
The kustomizations will be in the demo repos, i.e. demo2-repo and demo3-repo.

## Demo recording

You can watch the internal [recording here](https://drive.google.com/file/d/1xokDKJa_6smYE1TipTDtuTOi2sQvQLqA/view?usp=sharing)

TODO: record a customer version of the demo that can be used in case SAs are unavailable to do it in person.
