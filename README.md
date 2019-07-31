# Instrumenting Java Using Init Containers in Kubernetes

## Purpose
To demonstrate how a Java application can be instrumented with New Relic APM using an init container methodology.  The init container methodology does not require ANY changes to the application to be instrumented's container.  There are a host of articles on the Internet describing init containers and their benefits, but a good place to start is the Kubernetes documenations (https://kubernetes.io/docs/concepts/workloads/pods/init-containers/).

Included New Relic Java APM agent is at version 5.3.0.

## Prerequisites
- MacOS and/or Linux
- Docker (https://docs.docker.com/install/)
- Docker Hub account
- Homebrew (https://brew.sh/)

## Install and test Minikube (https://github.com/kubernetes/minikube)
1. `brew install hyperkit`
2. `curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-hyperkit && sudo install -o root -g wheel -m 4755 docker-machine-driver-hyperkit /usr/local/bin/`
3. `brew cask install minikube`
4. `minikube config set vm-driver hyperkit`
5. `minikube start --cpus 2 —-memory 4096`
6. `minikube dashboard` (should open a window in your default web browser)
7. `kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080`
8. `kubectl expose deployment hello-minikube --type=NodePort` (expose the service as a NodePort)
9. `minikube service hello-minikube` (launch the service in your default browser)

## Deploy an application to K8s that will be monitored by New Relic APM
1. `git clone https://github.com/danperovich/secrets-treasurehunt`
2. `cd secrets-treasurehunt`
3. `eval $(minikube docker-env)`
4. `docker build . -t treasurehunt`
5. `kubectl create -f ./treasurehunt`
6. `minikube service treasurehunt-entrypoint` (launches your default web browser)

## Delete both application deployments from K8s
1. `kubectl delete -f ./treasurehunt`
2. `kubectl delete all --selector run=hello-minikube`

## Build the New Relic Java APM  init container
1. `cd nr_container`
2. Review the contents of the Dockerfile to understand how the init/sidecar container is built
3. `docker build -t danperovich/newrelic-java-apm-sidecar:latest .` (replace ‘danperovich’ with your Docker Hub username)
4. `docker push danperovich/newrelic-java-apm-sidecar:latest`

## Modify the deployment descriptor to use the init container and override the default APM configuration values with those specific to this application/service
1. `cd ..`
2. `cp deployment.yaml.init.yaml treasurehunt/deployment.yaml` (it is okay to overwrite the original yams file, a backup is stored in the project home directory)
3. `vi treasurehunt/deployment.yaml`
4. Insert your New Relic APM license key string (line 30)
5. Update the image name on line 49 to reflect your Docker Hub username
6. Save and exit

## Redeploy app with new deployment container init container information
1. `kubectl create -f ./treasurehunt`
2. `minikube service treasurehunt-entrypoint` (launches your default web browser)
3. Verify application is reporting into New Relic APM

## Helful minikube and kubectl commands
- Delete all treasurehunt assets in K8s
  - `kubectl delete -f ./treasurehunt`
- Stop minikube
  - `minikube stop`
- Delete minikube cluster
  - `minikube delete`
