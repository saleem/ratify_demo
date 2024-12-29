# Ratify Demo

Based on the [online Ratify tutorial](https://ratify.dev/docs/quick-start/).

This list the steps I needed to do to run the Ratify demo on my Apple-silicon (M2) powered Macbook running macOS Sequoia 15.2 (24C101).

Some rather obvious prereqs – e.g., installing [homebrew](https://brew.sh) – have been omitted.

# Prerequisites

## Docker

Run `brew install docker` to install.

Verify that docker is correctly working by downloading a couple of images and running the corresponding containers. These two images satisfy the need:

### a. hello-world

```bash
docker run hello-world
```

This should print "Hello World" and some other reassuring text on the shell.

### b. hello-web

```bash
 docker run -d --rm --name hello-web -p 8080:8000 crccheck/hello-world
```

Verify that this gives you a web app by visiting http://localhost:8080 and/or running `curl http://localhost:8080` on the shell. (Borrowed from https://github.com/crccheck/docker-hello-world.)


## Kubernetes CLI

Run `brew install kubernetes-cli` to install.

Run `kubectl version` to verify version; must be `v1.20` or higher.

## Minikube

Run `brew install minikube` to install.

Run `minikube version` to verify version; `v1.34.0` works at this time (January 2025).

## OPA

Run `brew install opa` to install.

Run `opa version` to verify version; must be ...

## Helmfile

Run `brew install helmfile` to install.

Run `helmfile version` to verify verion; must be `v0.14` or higher.

# Steps for Ratify demo

## 1. Start Minikube

i. Download and run a simple web app via docker:

Start and monitor Minikube

```bash
minikube start

# This opens up a browser with the minikube dashboard; running this in the background allows us to kill the command once it's done without losing the dashboard
minikube dashboard &
```

(Borrowed from https://kubernetes.io/docs/tutorials/hello-minikube/.)


## 2. Install Gatekeeper, Ratify, and Constraints

```bash
helmfile sync -f git::https://github.com/ratify-project/ratify.git@helmfile.yaml
```

## 3. Create the pod demo with a signed image


```shell
kubectl run demo --image=ghcr.io/deislabs/ratify/notary-image:signed -n default

kubectl get pods demo -n default
```

This _SHOULD_ show you the one running pod: `demo`.

## 4. Try to deploy a pod with an unsigned image

```bash
kubectl run demo1 --image=ghcr.io/deislabs/ratify/notary-image:unsigned -n default
```

This _SHOULD_ generate an error similar to this:

```bash

Error from server (Forbidden): admission webhook "validation.gatekeeper.sh" denied the request: [ratify-constraint] Subject failed verification: ghcr.io/deislabs/ratify/notary-image@sha256:17490f90c4f278d4314a1ccbc407fc8fd00fb45303589b8cc7f5174bc35554f4

```

**This "failure" demonstrates that Ratify is working: it prevents the deployment of an unsigned image.**

## 5. Uninstall and verify negative use case

```bash
helmfile destroy --skip-charts -f git::https://github.com/ratify-project/ratify.git@helmfile.yaml
```

The same command, to install an unsigned pod, works now:

``` bash
kubectl run demo1 --image=ghcr.io/deislabs/ratify/notary-image:unsigned -n default
```

this _SHOULD_ succeed with this message:

```bash
pod/demo1 created
```

**This "success" demonstrates that Ratify is no longer present (and, therefore, ineffective): the unsigned image was deployed in a pod.**


## 6. Cleanup

Remove all pods and stop minikube:

```bash

kubectl delete pod demo
kubectl delete pod demo1

minikube stop

```


# Reference

## Useful `kubectl` commands

```bash

#get names of pods
kubectl get pod

# get all pods' names, status, and nodes
kubectl get pod -o=custom-columns=NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName --all-namespaces

```

## Useful `docker` commands

```bash

#list images
docker images

#remove an image
docker rmi <repository>

```
