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


# Install 

## k8see-exporter

Need :

* Redis server

Install :

```
kubectl apply -f https://raw.githubusercontent.com/sgaunet/k8see-deploy/main/manifests/k8see-exporter/ns.yaml
kubectl apply -f https://raw.githubusercontent.com/sgaunet/k8see-deploy/main/manifests/k8see-exporter/role-binding.yaml
kubectl apply -f https://raw.githubusercontent.com/sgaunet/k8see-deploy/main/manifests/k8see-exporter/role.yaml
kubectl apply -f https://raw.githubusercontent.com/sgaunet/k8see-deploy/main/manifests/k8see-exporter/sa.yaml
wget https://raw.githubusercontent.com/sgaunet/k8see-deploy/main/manifests/k8see-exporter/k8see-exporter-cm.yaml
# edit k8see-exporter-cm.yaml to configure redis access
kubectl apply -f k8see-exporter-cm.yaml
```


## k8see-importer

Need access :

* to the redis server (same as k8see-exporter of course)
* to a postgresql database (the user should have the rights to create tables)

If you use the docker image, you should configure the applicaiton with environement variable.

```
DBHOST: svc-postgres
DBPORT: "5432"
DBUSER: postgres
DBPASSWORD: password
DBNAME: mydb

REDIS_HOST: svc-redis
REDIS_PORT: "6379"
REDIS_STREAM: k8see-exporter

LOGLEVEL: "debug"
```

Otherwise, with binaries, you can specify a confiuration file (option -f) :

```
dbhost: localhost
dbport: "25432"
dbuser: postgres
dbpassword: password
dbname: mydb
loglevel: debug
redis_host: ""
redis_port: "6379"
redis_password: ""
redis_stream: k8sevents2pg
```

## k8see-webui

Can be outside the kubernetes cluster, it will query the postgreSQL database. Binaries and docker images are available.
Docker image is named: sgaunet/k8see-webui:0.1.0-beta1


If you use the docker image, you should configure the applicaiton with environement variable.

```
DBHOST: svc-postgres
DBPORT: "5432"
DBUSER: postgres
DBPASSWORD: password
DBNAME: mydb
```

Otherwise, with binaries, you can specify a confiuration file (option -f) :

```
db:
  host: localhost
  port: 25432
  user: postgres
  password: password
  dbname: mydb
```


# For tests

In the folder manifests/tests, you can setup a kubernetes test cluster with the whole stack (no persistence of data). Everything is deployed in the default namespace. As there is no persistence of data, the order is important, because it's the app k8see-importer that will initialize SQL tables.