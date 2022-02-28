# Ornikar technical test

https://github.com/ornikar/cloud-interview

## Kubernetes and ingress controller
### 1. Install Minikube

Well, just went on the Minikube [installation page](https://minikube.sigs.k8s.io/docs/start/) and issued the following commands :

```
brew install minikube
minikube start
```

> NOTE : I execute Minikube with the Docker driver previously installed following [this doc](https://docs.docker.com/desktop/mac/install/)
> Just bought a Mac some weeks ago and did not use it so much !

```
% minikube start

😄  minikube v1.25.2 on Darwin 12.2 (arm64)
✨  Automatically selected the docker driver
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.23.3 preload ...
    > preloaded-images-k8s-v17-v1...: 419.07 MiB / 419.07 MiB  100.00% 2.35 MiB
    > gcr.io/k8s-minikube/kicbase: 343.12 MiB / 343.12 MiB  100.00% 1.87 MiB p/
🔥  Creating docker container (CPUs=2, Memory=1988MB) ...
🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
🌟  Enabled addons: storage-provisioner, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
% 
```

```
% k get all --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-64897985d-ftb2k            1/1     Running   0          7m40s
kube-system   pod/etcd-minikube                      1/1     Running   0          7m53s
kube-system   pod/kube-apiserver-minikube            1/1     Running   0          7m54s
kube-system   pod/kube-controller-manager-minikube   1/1     Running   0          7m55s
kube-system   pod/kube-proxy-2x95h                   1/1     Running   0          7m41s
kube-system   pod/kube-scheduler-minikube            1/1     Running   0          7m54s
kube-system   pod/storage-provisioner                1/1     Running   0          7m52s

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  7m55s
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   7m54s

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   7m54s

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns   1/1     1            1           7m54s

NAMESPACE     NAME                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-64897985d   1         1         1       7m41s
% 
```

Sounds great :)

### Deploy Traefik as Ingress Controller

Sounds like Ornikar likes Helm. Great !

```
% brew install helm
% helm version
version.BuildInfo{Version:"v3.8.0", GitCommit:"d14138609b01886f544b2025f5000351c9eb092e", GitTreeState:"clean", GoVersion:"go1.17.6"}
```

Install Traefik with Helm : https://github.com/traefik/traefik-helm-chart

````
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install traefik traefik/traefik -n kube-system

k get po
NAME                               READY   STATUS    RESTARTS      AGE
coredns-64897985d-wkh4h            1/1     Running   0             59m
etcd-minikube                      1/1     Running   0             59m
kube-apiserver-minikube            1/1     Running   0             59m
kube-controller-manager-minikube   1/1     Running   0             59m
kube-proxy-xcnb2                   1/1     Running   0             59m
kube-scheduler-minikube            1/1     Running   0             59m
storage-provisioner                1/1     Running   1 (59m ago)   59m
traefik-68cc69f688-c8skf           1/1     Running   0             4m37s

k get svc
NAME       TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
kube-dns   ClusterIP      10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP       59m
traefik    LoadBalancer   10.99.28.72   <pending>     80:31529/TCP,443:30136/TCP   4m43s
````

Dashboard :
kubectl port-forward $(kubectl get pods --selector "app.kubernetes.io/name=traefik" --output=name -n kube-system) 9000:9000 -n kube-system

Browser : http://127.0.0.1:9000/dashboard/#/

> To be honest, I did not succeed to deploy the apps ingresses with `traefik`. 
> Instead, I installed nginx in Minikube : `minikube addons enable ingress`.


## Containerize applications

### Hello app

Looking at this app... Definitely a NodeJS app (Express !).

So, I'll use this [Docker Node image](https://hub.docker.com/_/node).

Here is the Dockerfile :
```
FROM node
WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn --version
ADD src ./src
RUN yarn install
ENTRYPOINT ["yarn", "run", "start"]

```

For production purpose, I would advice using a Docker multistage build where you'll
- first build your NodeJs binary using the Node image
- finally use a Nginx or Httpd image containing the preceding build.

Build : `docker build -t stockersky/ornikar-hello .`
push to Dockerhub : `docker push stockersky/ornikar-hello`

### World app

Sounds like a php app (composer).
At first, I tried with a [composer docker image](https://hub.docker.com/_/composer).
But had those errors :
```
> [4/4] RUN composer install:
#8 0.290 Installing dependencies from lock file (including require-dev)
#8 0.290 Verifying lock file contents can be installed on current platform.
#8 0.295 Your lock file does not contain a compatible set of packages. Please run composer update.
#8 0.295 
#8 0.295   Problem 1
#8 0.295     - monolog/monolog is locked to version 2.0.1 and an update of this package was not requested.
#8 0.295     - monolog/monolog 2.0.1 requires php ^7.2 -> your php version (8.1.3) does not satisfy that requirement.
#8 0.295 
```

Well, I don't know much about php but it this image has a php version > 8. 
So, I went with a base php image with specific version of '7' and installed `composer` inside it.

Here is the Dockerfile :

````
FROM php:7-fpm

WORKDIR /composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv composer.phar /usr/local/bin/composer

WORKDIR /app
COPY composer.json composer.lock ./
RUN apt-get update && apt-get install -y git
RUN composer install

ADD public ./public
ENTRYPOINT ["composer", "run", "dev:start"]
````

Build : `docker build -t stockersky/ornikar-world .`
push to Dockerhub : `docker push stockersky/ornikar-world`

## Package applications

Package app with [Helm](https://helm.sh/)

### Create the Helm Chart

`helm create ornikar-helm-chart`

### Values files

For this project, the two micro-services use the same Chart but have their own Values files.
This way, each micro-service `hello` and `world` can be create / updated / uninstalled on their own.

The whole Helm thing is located in the directory `ornikar-helm-chart` of this project.

- `hello-values.yaml` contains values for app `hello`
- `world-values.yaml` contains values for app `world`


Usually, I would locate those files in the app repo, not in the Chart repo.

## Deploy applications

### hello

`helm upgrade --install ornikar-hello ./ornikar-helm-chart/. -f ./ornikar-helm-chart/hello-values.yaml`


### world

`helm upgrade --install ornikar-world ./ornikar-helm-chart/. -f ./ornikar-helm-chart/world-values.yaml`

### some outputs

````
k get ingress
NAME                               CLASS   HOSTS         ADDRESS        PORTS   AGE
ornikar-hello-ornikar-helm-chart   nginx   ornikar.dev   192.168.49.2   80      6h4m
ornikar-world-ornikar-helm-chart   nginx   ornikar.dev   192.168.49.2   80      60m
````

````
helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS  CHART                    APP VERSION
ornikar-hello   default         7               2022-02-28 00:09:50.452959 +0100 CET    deployedornikar-helm-chart-0.1.0 1.16.0     
ornikar-world   default         3               2022-02-28 00:31:33.010568 +0100 CET    deployedornikar-helm-chart-0.1.0 1.16.0   
````

## final test

Port-forward Nginx Ingress Controler to localhost :

`k port-forward svc/ingress-nginx-controller 8080:80 -n ingress-nginx`

test :

`curl http://ornikar.dev:8080/hello` --> Display Hello

`curl http://ornikar.dev:8080/world` --> ERROR


## NOTES

Php server executed through composer seems to have a sort of timeout. It never passed Readiness / liveness probes.

I know the Kube Service associated with it worked as I got the "World" displayed while testing the Service. But even directly in Docker it does not work properly.
I guess, I'd have to dig in Php. But, unfortunately, I don't have the time for that :( .

Cheers :) !
