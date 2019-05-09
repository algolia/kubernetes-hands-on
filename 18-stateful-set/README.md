# Other topics

## Introduction

In this section you will get an overview of others k8s useful features, in order of complexity.

## Namespace

`Namespaces` is the way to support multiple virtual clusters in k8s.

They are intended for use in environments with many users spread across multiple teams, or projects. For clusters with a few to tens of users, you should not need to create or think about `namespaces` at all. Start using `namespaces` when you need the features they provide.

By default, all objects are in the `default` namespace. There is a "hidden" `namespace` where k8s runs services for itself.
Try:

```bash
$ kubectl get namespace
NAME          STATUS    AGE
default       Active    56d
kube-public   Active    56d
kube-system   Active    56d
```

```bash
$ kubectl get all --namespace=kube-system

[lot of stuff]
```

## `kubeval`

It is a tool to validate your k8s YAML files: <https://github.com/garethr/kubeval>

The easiest integration is with `docker run`, if you files are in the directory `kubernetes`

```bash
docker run -it -v `pwd`/kubernetes:/kubernetes garethr/kubeval kubernetes/**/*
```

## Helm

It is a package manager for k8s: <https://helm.sh/>.
It contains multiple, ready to use, k8s manifest for projects, for example [mysql](https://github.com/helm/charts/tree/master/stable/mysql)

## Stateful Set

Like a `Deployment`, a `StatefulSet` manages Pods that are based on an identical container spec. Unlike a `Deployment`, a `StatefulSet` maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

`StatefulSets` are valuable for applications that require one or more of the following.

* Stable, unique network identifiers, ex: distributed system, like ElasticSearch
* Stable, persistent storage, ex: MySQL
* Ordered, graceful deployment and scaling
* Ordered, automated rolling updates, ex: MySQL Master+Slave

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

## Exercises

1. Install `helm`, and use it to install [`redis`](https://github.com/helm/charts/tree/master/stable/redis) in your minikube
2. Configure a stateful set for nginx with a HPA at 1% CPU, in a namespace `staging`

## Clean up

```bash
kubectl delete statefulset,deployment,service,pod --all
```
