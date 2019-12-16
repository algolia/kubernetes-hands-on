# Other topics

## Introduction

In this section you will get an overview of others Kubernetes useful features, in order of complexity.

## Namespace

`Namespaces` is the way to support multiple virtual clusters in Kubernetes.

They are intended for use in environments with many users spread across multiple teams, or projects. For clusters with a few to tens of users, you should not need to create or think about `namespaces` at all. Start using `namespaces` when you need the features they provide.

By default, all objects are in the `default` namespace. There is a "hidden" `namespace` where Kubernetes runs services for itself.
Try:

```sh
$ kubectl get namespace
NAME          STATUS    AGE
default       Active    56d
kube-public   Active    56d
kube-system   Active    56d
```

```sh
$ kubectl get all --namespace=kube-system
[lot of stuff]
```

## `kubeval`

It is a tool to validate your Kubernetes YAML files: https://github.com/garethr/kubeval

The easiest integration is with `docker run`, if you files are in the directory `kubernetes`

```sh
docker run -it -v `pwd`/kubernetes:/kubernetes garethr/kubeval kubernetes/**/*
```

## Helm

It is a package manager for Kubernetes: https://helm.sh/.
It contains multiple, ready to use, Kubernetes manifest for projects, for example [mysql](https://github.com/helm/charts/tree/master/stable/mysql)

## Kube state metrics

[Kube State Metrics](https://github.com/kubernetes/kube-state-metrics) is a service you can install on your Kubernetes clusters to get metrics from its state. It's very useful for production cluster as you can measure and put alerts on the state of your applications. Like when do you have pod evictions, are your deployment fully deployed, etc.

## Clean up

```sh
kubectl delete statefulset,deployment,service,pod --all
```
