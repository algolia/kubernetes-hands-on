# Kubernetes Hands on

# Sections

0. Prerequisites
1. What this is *not*
2. What is kubernetes? What is it used for?
3. Base glossary
4. The base building block: `pod`
5. Naming things: `label` & `annotation`
6. Deploying my first application: `deployment`
7. Accessing my first application: `service`
8. Running a background process: `cronjob`
9. Keeping a state between pods: `stateful set`
10. Other topics

## Prerequisites

* brew: https://brew.sh/

```bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

* docker: https://docs.docker.com/docker-for-mac/install/

```bash
$ open https://download.docker.com/mac/stable/Docker.dmg
```

* minikube: https://github.com/kubernetes/minikube

```bash
$ brew cask install minikube
$ minikube start
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.7", GitCommit:"0c38c362511b20a098d7cd855f1314dad92c2780", GitTreeState:"clean", BuildDate:"2018-08-20T10:09:03Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-26T16:44:10Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

## What this is *not*

This is not a hands on on how to install/manage/deploy a k8s cluster.
This is neither a hands on to understand how kubernetes is working internally.
If this topic interests you see [Kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way).

## What is kubernetes? What is it used for?

## Glossary
* **yml/yaml**

A markup language that relies on spaces & tabulation. All k8s configuration is written using yaml.

You will feel the pain of missing tabs & spaces.
Feel free to use a linter, http://www.yamllint.com/.

* **container**

Containers are an abstraction at the app layer that packages code and dependencies together

* **(container) image**

A lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

* **docker**

A runtime for containers, https://www.docker.com

* **kubectl**

The standard cli to interact with k8s, we will use it a lot.

* **(kubernetes) kind**

resource this object represents

* **(kubernetes) cluster**
* **(kubernetes) master**

The Master is responsible for managing the cluster. The master coordinates all activities in your cluster, such as scheduling applications, maintaining applications’ desired state, scaling applications, and rolling out new updates.

Kubernetes master automatically handles scheduling the pods across the Nodes in the cluster. The Master’s automatic scheduling takes into account the available resources on each Node.

* **(kubernetes)** node:

## The base building block: `pod`

See the dedicated [README](04-pods).

## Naming things: `label` & `annotation`

See the dedicated [README](05-label-annotation).

## Deploying my first application: `deployment`

See the dedicated [README](06-deployment).

## Accessing my first application: `service`

See the dedicated [README](07-service).

## Running a background process: `cronjob`

See the dedicated [README](08-cronjob).

## Keeping a state between pods: `stateful set`

See the dedicated [README](09-stateful-set).

## Other topics

See the dedicated [README](10-other-topics).

## Links

* http://kubernetesbyexample.com/
* https://kubernetes.io/docs/home/
* https://kubernetes.io/docs/reference/kubectl/cheatsheet/
* https://docs.google.com/presentation/d/18_o45N86XPnkv4YMzhZHsPkMRB7As7eI6UgvXrQ6aG0/edit#slide=id.g435d19b4fa_0_9
* https://hub.docker.com/r/mhausenblas/simpleservice/

