# k8see-deploy

Contains all manifests to deploy the component of k8sEventsExporter in kubernetes 

* Folder **manifests/k8see-exporter** contains manifest to deploy k8see-exporter in k8s
* Folder **manifests/tests** contains kind configuration and manifest to deploy the whole stack in kubernetes (easy to test)

# Architecture

## k8see-exporter

Need to be launched in kubernetes, it will connect to kubernetes API and write in a redis stream the events of kubernetes. Manifests are in foler manifests/k8see-exporter.
Docker image is named : sgaunet/k8see-exporter:0.1.0

## k8see-importer

Can be outside of the kubernetes cluster, use the docker image or the binary. It will read the redis stream and write events in postgreSQL database.
Docker image is named: sgaunet/k8see-importer:0.1.0

## k8see-webui

Can be outside the kubernetes cluster, it will query the postgreSQL database. Binaries and docker images are available.
Docker image is named: sgaunet/k8see-webui:0.1.0-beta1

# For tests

In the folder manifests/tests, you can setup a kubernetes test cluster with the whole stack (no persistence of data). Everything is deployed in the default namespace. As there is no persistence of data, the order is important, because it's the app k8see-importer that will initialize SQL tables.