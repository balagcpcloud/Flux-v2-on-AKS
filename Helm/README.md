# Helm

This quick guide has two parts:

- Using Helm
- Creating a Helm chart

## Using Helm

Requirements:
- bash
- Helm v3 (`brew install helm`)
- kubectl (and working kube config)

```bash
# Check helm version; mine was 3.9.x
helm version

# Helm charts live in repositories; add the repository for super-api
helm repo add super-api https://gbaeke.github.io/helm-chart/

# List repositories; super-api should be listen
helm repo list

# Search the repository
helm search repo super-api

# The repository has one chart; it has a chart version, an appVersion and a description
# example: super-api/super-api 1.0.0   1.0.3  A Helm chart for super-api
# Let's install the chart and call the release myapp; install in a namespace and create it
# if it does not exist

helm install myapp super-api/super-api --namespace super-api --create-namespace

# list Helm installations in namespace
helm list --namespace super-api

# check if pods were deployed; should result in one pod
# the name of the pod includes myapp (release name) and super-api (chart name)
# the names of deployments & pods depend on how the chart was created
kubectl get pods --namespace super-api

# run helm test (-n is the same as --namespace)
# you can run this command because the super-api chart has a test defined
# the test should return success
helm test myapp -n super-api

# check pods again; there should be a completed pod that helm used to perform the test
# the test simply ran a curl/wget command to verify that super-api returns a response
kubectl get pods --namespace super-api

# Let's install version 1.0.7 of the application  by setting a user-defined value
# We use helm upgrade instead of helm install
# --install to install the chart if the chart was not installed yet
# --set is used to set the value of image.tag to 1.0.7; the chart was created to use that value as the image tag

helm upgrade --install myapp super-api/super-api --namespace super-api --set image.tag=1.0.7

# run helm list again; we are now at revision 2 of myapp
helm list -n super-api

# get more information about the revisions with helm history
# you will see that revision two is the deployed one
# do not be misled by the APP VERSION; that is the default set in the chart and has no relation
# with the actual installed version (1.0.7)
helm history myapp -n super-api

# get information about the latest release; first get ALL information about the release
# ALL INFORMATION = metadata, user-supplied values, computed values, the manifest, hooks, notes.txt 
helm get all myapp -n super-api

# to only get user supplied values
helm get values myapp -n super-api

# to only get the manifest that is generated from the templates in the chart
# the chart developer decides which templates to include and how to fill them with
# computed values; computer values are the combination of default values & user supplied values
helm get manifest  myapp -n super-api

# to only get notes; notes are defined by the chart developer by included a notes.txt in the templates
# and optionally show computed values in the notes (.e.g. deployment name, pod names, ...)
helm get notes myapp -n super-api

# get information about a specific revision
# e.g, get values for the first revision which results in null (no values supplied)
helm get values myapp -n super-api --revision 1

# rollback to revision 1
helm rollback myapp 1 -n super-api

# helm history will now show 3 revisions; the 3rd revision is the rollback to 1
helm history myapp -n super-api

# helm keeps history information as secrets in the namespace
# run the command below to see three secrets of type helm.sh/release.v1
kubectl get secrets -n super-api

# let's uninstall the chart but keep the history
helm uninstall myapp -n super-api --keep-history

# history will still show 3 releases (the secrets are still in the namespace)
helm history myapp -n super-api

# nothing will be shown if you run helm list
helm list -n super-api

# but you can still rollback
helm rollback myapp 2 -n super-api

# helm history will show revision 3 uninstalled and revision 4 as rollback to 3
# note: if you do not use --keep-history during uninstall you cannot do a rollback afterwards
helm history myapp -n super-api

```

