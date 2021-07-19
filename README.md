# Pega on Openshift
## Background
This repository is an unofficial guide to installing Pega on an OpenShift cluster. It is a work in progress and should be used as a general guide subject to change.

## Prerequisites
- OpenShift 4.7.x cluster with a minimum of 3 worker nodes with 32GB RAM
- Valid credentials to Pega images **pega-docker.downloads.pega.com** 
- The *prereqs* directory contains a template to create the required postgresql database for the Pega installation.

## Installation Steps

1. Create postgresql database.
```
oc new-project pega-db
oc new-app prereqs/pega-postgres_template.json

```

2. Create namespaces for components and grant scc's for containers to run
```
oc new-project pegaaddons
oc adm policy add-scc-to-user anyuid -z default --as system:admin
oc new-project pegabackingservices
oc adm policy add-scc-to-user anyuid -z default --as system:admin
oc new-project mypega
oc adm policy add-scc-to-user anyuid -z default --as system:admin
```

3. Add helm repo
```
helm repo add pega https://pegasystems.github.io/pega-helm-charts
```

4. Run backing services chart
```
helm install backingservices pega/backingservices --namespace pegabackingservices --values values/backingservices.yaml
```
NOTE: This currently errors out with invalid credentials against pega registry

5. Run addons chart
```
helm install addons pega/addons --namespace pegaaddons --values values/addons.yaml
```
NOTE: This currently fails readiness probes for pod

6. Run pega chart
```
helm install mypega pega/pega --namespace mypega --values values/pega.yaml
```
NOTE: Installer job completes but dependent jobs (batch, web, stream) timeout waiting for components like pega-search, etc. to start