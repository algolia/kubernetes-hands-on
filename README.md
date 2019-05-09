# Kubernetes Hands on

1. [Prerequisites](#prerequisites)
1. [What it is not](#what-it-is-not)
1. [What is kubernetes? What is it used for?](#what-is-kubernetes-what-is-it-used-for)
1. [Glossary](#glossary)
1. [The base building block: pod](#the-base-building-block-pod)
1. [Naming things: label and annotation](#naming-things-label-and-annotation)
1. [Deploying my first application: deployment](#deploying-my-first-application-deployment)
1. [Accessing my first application: service](#accessing-my-first-application-service)
1. [Running a background process: cronjob](#running-a-background-process-cronjob)
1. [Secrets](#secrets)
1. [Liveness and readiness probes, and how it impacts your pods](#liveness-and-readiness-probes,-and-how-it-impacts-your-pods)
1. [Resources, and how it impacts the scheduling](#resources,-and-how-it-impacts-the-scheduling)
1. [Improve the availability of your application: affinity and anti-affinity](#affinity-and-anti-affinity)
1. [Improve the availability of your application: pod disruptions budget](#pdb)
1. [Improve the elasticiy of your applications: HPA, VPA](#hpa-vpa)
1. [Sidecar containers: what, why, and how](#sidecar-containers-what,-why,-and-how)
1. [Running a stateful application: volumes](#running-a-stateful-application-volumes)
1. [Running a stateful application: stateful-sets](#running-a-stateful-application-stateful-sets)
1. [Controllers: what, why, and how](#controllers-what,-why,-and-how)
1. [Operators and CRDs: what, why, and how](#operators-and-crds-what,-why,-and-how)
1. [RBAC](#rbac)
1. [Good practices](#good-practices)
1. [Links](#links)

## License

This hands-on in under the [CC BY-NC-SA](./LICENSE) license.

![CC BY-NC-SA](https://licensebuttons.net/l/by-nc-nd/3.0/88x31.png)

## Prerequisites

* brew: <https://brew.sh/>

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

* docker: <https://docs.docker.com/docker-for-mac/install/>

```bash
open https://download.docker.com/mac/stable/Docker.dmg
```

* VirtualBox: <https://www.virtualbox.org/wiki/Downloads>

* minikube: <https://github.com/kubernetes/minikube>

```bash
$ brew cask install minikube

$ minikube start
[...]
🏄  Done! Thank you for using minikube!

$ minikube addons enable ingress
✅  ingress was successfully enabled

$ kubectl config current-context
minikube
```

### (Optional) If you feel adventurous, only for macOS

You can try another lighter VM layer than Virtualbox

* docker-machine-driver-hyperkit: <https://github.com/moby/hyperkit>

```bash
brew install docker-machine-driver-hyperkit
```

And start minikube with

```bash
minikube start --vm-driver=hyperkit
```

If you have any issues:

```bash
rm -rf ~/.minikube/
```

And start minikube without hyperkit

```bash
minikube start
```

### Completion

If you are using zsh, you can add to your `.zshrc` file this to have autocomplete of `kubectl`:

```bash
if [ $commands[kubectl] ]; then
  source <(kubectl completion zsh)
fi
```

## What this is and what this is *not*

### What this is

This is a hands on to start with using kubernetes (k8s). It starts from the basics and moves up in complexity.
At the end of this hands on you should be able to deploy an API in kubernetes that is accessible from the outside.

### What this is *not*

This is not a hands on on how to install/manage/deploy a k8s cluster.
This is neither a hands on to understand how kubernetes is working internally.
If this topic interests you, see [Kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way).

## What is kubernetes? What is it used for?

Kubernetes is an open source system for managing containerized applications across multiple hosts, providing basic mechanisms for deployment, maintenance, and scaling of applications.

Kubernetes has a number of features. It can be thought of as:

* a container platform,
* a microservices platform,
* a portable cloud platform and a lot more.

Kubernetes provides a container-centric management environment. It orchestrates computing, networking, and storage infrastructure on behalf of user workloads. This provides much of the simplicity of Platform as a Service (PaaS) with the flexibility of Infrastructure as a Service (IaaS), and enables portability across infrastructure providers.

## Glossary

* **yml/yaml**

A markup language that relies on spaces & tabulation. All k8s configuration is written using yaml.

You will feel the pain of missing tabs & spaces.
Feel free to use a linter, <http://www.yamllint.com/.>

* **container**

Containers are an abstraction at the app layer that packages code and dependencies together.

* **(container) image**

A lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings.

* **docker**

A software technology providing operating-system-level virtualization also known as containers.

Docker uses the resource isolation features of the Linux kernel such as cgroups and kernel namespaces, and a union-capable file system such as OverlayFS and others to allow independent “containers” to run within a single Linux instance, avoiding the overhead of starting and maintaining virtual machines (VMs).

* **kubectl**

The standard cli to interact with k8s, we will use it a lot.

* **minikube**

A local kubernetes, useful for testing. We will use it during this hands on.

* **manifest**

Kubernetes configuration files are called `manifest`. In reference to the `manifest` of a ship: A list or invoice of the passengers or goods being carried by a commercial vehicle or ship (from [wiktionary](https://en.wiktionary.org/wiki/manifest#Noun)).

* **(kubernetes) objects**

Kubernetes contains a number of abstractions that represent the state of your system: deployed containerized applications and workloads, their associated network and disk resources, and other information about what your cluster is doing. These abstractions are called `objects` and represented by a `kind` in the Kubernetes API.

* **(kubernetes) cluster**

A set of machines, called nodes, that run containerized applications managed by Kubernetes.

A cluster has several worker nodes and at least one master node.

* **(kubernetes) master**

The Master is responsible for managing the cluster. The master coordinates all activities in your cluster, such as scheduling applications, maintaining applications’ desired state, scaling applications, and rolling out new updates.

Kubernetes master automatically handles scheduling your services across the Nodes in the cluster. The Master’s automatic scheduling takes into account the available resources on each Node.

* **(kubernetes) node**

A node is a worker machine in Kubernetes.

A worker machine may be a VM or physical machine, depending on the cluster. It has the Services necessary to run the services and is managed by the master components. The Services on a node include Docker, `kubelet` and `kube-proxy`.

## The base building block: `pod`

See the dedicated [README](05-pods).

## Naming things: `label` and `annotation`

See the dedicated [README](06-label-annotation).

## Deploying my first application: `deployment`

See the dedicated [README](07-deployment).

## Accessing my first application: `service`

See the dedicated [README](08-service).

## Running a background process: `cronjob`

See the dedicated [README](09-cronjob).

## Secrets

See the dedicated [README](10-secrets).

## Liveness and readiness probes, and how it impacts your pods

See the dedicated [README](11-probes).

## Resources, and how it impacts the scheduling

See the dedicated [README](12-resources).

## Affinity and anti-affinity

See the dedicated [README](13-affinity-anti-affinity).

## PDB

See the dedicated [README](14-pdb).

## HPA, VPA

See the dedicated [README](15-hpa-vpa).

## Sidecar containers: what, why, and how

See the dedicated [README](16-sidecar-containers).

## Running a stateful application: `volumes`

See the dedicated [README](17-volumes).

## Running a stateful application: `stateful sets`

See the dedicated [README](18-stateful-sets).

## Controllers: what, why, and how

See the dedicated [README](19-controllers).

## Operators and CRDs: what, why, and how

See the dedicated [README](20-operators).

## RBAC

See the dedicated [README](21-rbac).

## Good practices

See the dedicated [README](99-good-practices).

## Links

* <http://kubernetesbyexample.com/>
* <https://kubernetes.io/docs/home/>
* <https://kubernetes.io/docs/reference/kubectl/cheatsheet/>
* <https://hub.docker.com/r/mhausenblas/simpleservice/>
