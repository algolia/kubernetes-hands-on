# Kubernetes Hands on

1. [Prerequisites](#prerequisites)
1. [What it is not](#what-it-is-not)
1. [What is Kubernetes? What is it used for?](#what-is-kubernetes-what-is-it-used-for)
1. [Glossary](#glossary)
1. [The base building block: pods](#the-base-building-block-pods)
1. [Naming things: labels and annotations](#naming-things-labels-and-annotations)
1. [Deploying your first application: deployment](#deploying-my-first-application-deployment)
1. [Accessing your first application: service](#accessing-my-first-application-service)
1. [Running a background process: cronjobs](#running-a-background-process-cronjobs)
1. [Secrets](#secrets)
1. [Liveness and readiness probes, and how it impacts your pods](#liveness-and-readiness-probes,-and-how-it-impacts-your-pods)
1. [Resources, and how it impacts the scheduling](#resources,-and-how-it-impacts-the-scheduling)
1. [Improving the availability of your application: affinity and anti-affinity](#affinity-and-anti-affinity)
1. [Improving the availability of your application: pod disruptions budget](#pdb)
1. [Improving the elasticity of your applications: HPA, VPA](#hpa-vpa)
1. [Sidecar containers: what, why, and how](#sidecar-containers-what,-why,-and-how)
1. [Running a stateful application: volumes](#running-a-stateful-application-volumes)
1. [Running a stateful application: stateful-sets](#running-a-stateful-application-stateful-sets)
1. [Controllers: what, why, and how](#controllers-what,-why,-and-how)
1. [Operators and CRDs: what, why, and how](#operators-and-crds-what,-why,-and-how)
1. [RBAC](#rbac)
1. [Other topics](#other-topics)
1. [Good practices](#good-practices)
1. [Links](#links)

## License

This hands-on course in under the [CC BY-NC-SA](./LICENSE) license.

![CC BY-NC-SA](https://licensebuttons.net/l/by-nc-nd/3.0/88x31.png)

## Prerequisites

* Homebrew: <https://brew.sh/>

```sh
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

* Docker: <https://docs.docker.com/docker-for-mac/install/>

```sh
open https://download.docker.com/mac/stable/Docker.dmg
```

* VirtualBox: <https://www.virtualbox.org/wiki/Downloads>

* minikube: <https://github.com/kubernetes/minikube>

Install minikube and the "ingress" and "metrics-server" addons:

```sh
$ brew install kubectl
[...]

$ brew cask install minikube
[...]

$ minikube start
[...]
🏄  Done! Thank you for using minikube!

$ minikube addons enable ingress
✅ ingress was successfully enabled

$ minikube addons enable metrics-server
✅  metrics-server was successfully enabled

$ kubectl config current-context
minikube
```

**Note:** the ingress addon is currently not supported on docker for Mac (see https://github.com/kubernetes/minikube/issues/7332).
As a workaround, you have to deploy minikube as a VM and not as a container (using Virtualbox or Hyperkit for example)

```sh
$ minikube start --vm=true --vm-driver=virtualbox
[...]
✨  Using the virtualbox driver based on user configuration
🔥  Creating virtualbox VM ...
```

If you did launch minikube already, the `--vm` flag may be ignored as minikube caches the previous config. If so you may want to delete and relaunch minikube (warning: it will delete your whole minikube setup)

```sh
$ minikube stop && minikube delete && minikube start --vm=true --vm-driver=virtualbox
[...]
💀  Removed all traces of the "minikube" cluster.
✨  Using the virtualbox driver based on user configuration
🔥  Creating virtualbox VM ...
```

### (Optional) If you feel adventurous, only for macOS

You can try another lighter virtual machine layer than Virtualbox

* HyperKit: <https://github.com/moby/hyperkit>

```sh
brew install docker-machine-driver-hyperkit
```

Then start minikube:

```sh
minikube start --vm-driver=hyperkit
```

If you're encountering any issues:

```sh
rm -rf ~/.minikube/
```

And start minikube without HyperKit:

```sh
minikube start
```

### Completion

If you are using Zsh, you can add the following to your `.zshrc` file to get autocomplete for `kubectl`:

```sh
if [ $commands[kubectl] ]; then
  source <(kubectl completion zsh)
fi
```

## What this course is and what it's *not*

### What this is

This is a hands-on course to get started with Kubernetes (Kubernetes). It starts with the basics and moves up in complexity.
At the end of this course, you should be able to deploy an API in Kubernetes that is accessible from the outside.

### What it's *not*

This is not a course on how to install, manage or deploy a Kubernetes cluster.
Neither is it a course to understand how Kubernetes works internally.
However, if you're interested in this topic, see [Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way).

## What is Kubernetes? What is it used for

Kubernetes is an open-source system for managing containerized applications across multiple hosts, providing basic mechanisms for deployment, maintenance, and scaling of applications.

Kubernetes has a number of features. It can be seen as:

* a container platform,
* a microservices platform,
* a portable cloud platform, and a lot more.

Kubernetes provides a container-centric management environment. It orchestrates computing, networking, and storage infrastructure on behalf of user workloads. This provides much of the simplicity of Platform as a Service (PaaS) with the flexibility of Infrastructure as a Service (IaaS), and enables portability across infrastructure providers.

## Glossary

* **YAML (yml)**

A markup language that relies on spaces and tabulations. All Kubernetes configuration is written using YAML.

You will feel the pain of missing tabs and spaces. Feel free to use a linter, such as <http://www.yamllint.com/>.

* **Container**

*Containers* are an abstraction at the app layer, which packages code and dependencies together.

* **(Container) image**

A lightweight, standalone, executable software package that includes everything you need to run an application: code, runtime, system tools, system libraries and settings.

* **Docker**

A software technology providing operating-system-level virtualization, also known as containers.

Docker uses the resource isolation features of the Linux kernel, such as cgroups and kernel namespaces, and a union-capable file system such as OverlayFS and others to allow independent “containers” to run within a single Linux instance. This avoids the overhead of starting and maintaining virtual machines (VMs).

* **kubectl**

The standard CLI to interact with Kubernetes. We use it a lot in this course.

* **minikube**

A local Kubernetes cluster, useful for testing. We use it a lot in this course.

* **Manifest**

Kubernetes configuration files are called *manifests*. This is a reference to the list or invoice of the passengers or goods being carried by a commercial vehicle or ship (from [wiktionary](https://en.wiktionary.org/wiki/manifest#Noun)).

* **(Kubernetes) objects**

Kubernetes contains a number of abstractions that represent the state of your system: deployed containerized applications and workloads, their associated network and disk resources, and other information about what your cluster is doing. These abstractions are called *objects*, and are represented by a *kind* in the Kubernetes API.

* **(Kubernetes) node**

A node is a worker machine in Kubernetes.

A worker machine may be a VM or physical machine, depending on the cluster. It has the necessary services to run the workloads and is managed by the master components. The services on a node include Docker, `kubelet` and `kube-proxy`.

* **(Kubernetes) cluster**

A set of machines, called nodes, that run containerized applications managed by Kubernetes.

A cluster has several worker nodes and at least one master node.

* **(Kubernetes) master**

The *master* is responsible for managing the cluster. It coordinates all activities in your cluster, such as scheduling applications, maintaining applications’ desired state, scaling applications, and rolling out new updates.

A Kubernetes master automatically handles the scheduling of your services across nodes in the cluster. The master’s automatic scheduling takes the available resources of each node into account.

## The base building block: pods

See the dedicated [README](05-pods).

## Naming things: labels and annotations

See the dedicated [README](06-label-annotation).

## Deploying my first application: deployment

See the dedicated [README](07-deployment).

## Accessing my first application: service

See the dedicated [README](08-service).

## Running a background process: cronjobs

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

## Running a stateful application: volumes

See the dedicated [README](17-volumes).

## Running a stateful application: stateful sets

See the dedicated [README](18-stateful-sets).

## Controllers: what, why, and how

See the dedicated [README](19-controllers).

## Operators and CRDs: what, why, and how

See the dedicated [README](20-operators).

## RBAC

See the dedicated [README](21-rbac).

## Other topics

See the dedicated [README](98-other-topics).

## Good practices

See the dedicated [README](99-good-practices).

## Links

* http://kubernetesbyexample.com/
* https://kubernetes.io/docs/home/
* https://kubernetes.io/docs/reference/kubectl/cheatsheet/
* https://hub.docker.com/r/mhausenblas/simpleservice/
