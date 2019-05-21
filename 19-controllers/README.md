# Controllers: what, why, and how

## Introduction

A controller is an application that watches and maintains a set of resources in a desired state.
Most of the previous resources we saw are in fact controllers, like the deployments.

To sum it up it is an application that looks like that:

```go
for {
  desired := getDesiredState()
  current := getCurrentState()
  makeChanges(desired, current)
}
```

We won't go into all the details of a controller. There are already a good resources online if you are interested.

## The basics of controllers

For this, we will use the example from the [Kubernetes patterns book](https://learning.oreilly.com/library/view/kubernetes-patterns/9781492050278/). The source code of the example we will use is available [here](https://github.com/k8spatterns/examples/tree/master/advanced/Controller).

We will write a controller that deletes pods based on a annotation in a `ConfigMap`

First let's create a [ConfigMap](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/). A `ConfigMap` is a list of key/values that can be shared between pods.

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configuration
  annotations:
    k8spatterns.io/podDeleteSelector: "app=nginx"
```

We will only use the `annotations` field in this config map. We created a custom annotation named "k8spatterns.io/podDeleteSelector". Our controller will act only on this annotation. Here we will delete pods with the label `app=nginx`.

Review and apply the manifest [01-configmap.yml](01-configmap.yml).

Next, let's create a deployment that will create pods for us, with the label `app=nginx`:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
```

Review and apply the manifest [02-deployment.yml](02-deployment.yml).

Now we need a controller to act on this `ConfigMap`. To simplify, an image containing the controller code is available on [docker hub](https://cloud.docker.com/u/elpicador/repository/docker/elpicador/kubernetes-controller). The controller [config-watcher-controller.sh](https://github.com/k8spatterns/examples/blob/master/advanced/Controller/config-watcher-controller.sh) is taken from the Kubernetes patterns github repository. For now don't look at it.

Review and apply the manifest [03-controller.yml](03-controller.yml). For now ignore the manifests `ServiceAccount` and `RoleBinding`.

Display the log of the container `expose-controller`, and the pods running. What can you see? Did you expect it?

In fact, the controller only reacts when a config map is modified. So modify the config map. See what happens.

## Deep dive in the controller code

Remember a controller is a loop doing:

```go
for {
  desired := getDesiredState()
  current := getCurrentState()
  makeChanges(desired, current)
}
```

You can look at the source code [here](https://raw.githubusercontent.com/ElPicador/kubernetes-controller/master/config-watcher-controller.sh).

In our case the `desired` is which pods should be deleted, so the content of the annotation `k8spatterns.io/podDeleteSelector`.
We get this by calling the Kubernetes API with the call:

```sh
curl -N -s "$base/api/v1/$ns/configmaps?watch=true" | while read -r event
```

The `current` is the pods actually running. Same, to get this we call the Kubernetes API:

```sh
local pods=$(curl -s ${base}/api/v1/${ns}/pods?labelSelector=${selector} | \
               jq -r .items[].metadata.name)
```

Fortunatelly, the API supports `labelSelector` so we can filter the pods easily.

Finally, the `makeChanges` is to delete all the pods matching the annotations:

```sh
exit_code=$(curl -s -X DELETE -o /dev/null -w "%{http_code}" ${base}/api/v1/${ns}/pods/${pod})
```

Again, we use the Kubernetes API for this

All the rest of the code is purely bash wrapping and `jq` magic to get the information we want.

## Around the controller

You've probably see that the manifests for our controller doesn't only contain a container with our code the `expose-controller`. It has a sidecar container `kubeapi-proxy`. This container is a proxy to the Kubernetes API, with all the security baked in.

The `ServiceAccount` and `RoleBinding` are here to declare which rights our pods needs when interracting with the Kubernetes API. We will see more details in the [RBAC chapter](../21-rbac).
For now think of the `ServiceAccount` as a user. And the `RoleBinding` as assigning rights to a specific user. Then in pod we specify a service account for it to use with the `serviceAccountName: expose-controller`.

## Clean up

```sh
kubectl delete deployment,configmap,serviceaccount,rolebinding --all
```

## Links

* https://engineering.bitnami.com/articles/a-deep-dive-into-kubernetes-controllers.html
