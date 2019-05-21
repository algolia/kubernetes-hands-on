# Resources, and how it impacts the scheduling

## Introduction

When you declare a `pod` you can declare what resources it will use. Those resources are how much CPU and RAM your pods will use.

Specifying those will help Kubernetes to schedule more efficently your pods on the available nodes.

For each resource you can define the `limits` and the `requests`.

## Resources definition

The CPU resource is measured in a number of CPU the pod will use for a given amount of time. It can be inferior to 0.
Specifying `0.5` (or `500m`, which means 500 millicpu), will give half of a CPU to your pod.

The RAM resource is measured in the number of bytes of RAM the pod will use.
Specifying `123Mi` (or `128974848`, which means 128974848 bytes), will give that many RAM to your pod.

## `requests` vs `limits`

Kubernetes let's you configure the `requests` and the `limits` for each resource.

They are put on at the container level:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-application
spec:
  containers:
  - name: my-application
    image: my-application:my-tag
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

Let's see in details each of those.

### `requests`

The `requests` is the number that will help Kubernetes schedule your pod on a node were the resources are available.

Let's take an example. You have 3 nodes:

* node 1 has `100m` CPU available, and `1Gi` RAM available
* node 2 has `1000m` CPU available, and `100Mi` RAM available

1. You start a pod with only a CPU request of `10m`. It can be scheduled on any node, so Kubernetes will take either of them.
1. You start a pod with only a CPU request of `500m`. It can be scheduled only on node 2. Node 1 has only `100m` CPU available.
1. You start a pod with a CPU request of `500m` and a RAM request of `500Mi`. Kubernetes will not be able to schedule your pod as no node has both requests at the same time. The pod will be in the state `Unschedulable`.

### `limits`

The `limits` is the maximum utilization of a resource Kubernetes will allow for a given pod. If your pod goes above this limit it will be restarted.

## Note on resources

On any computer, you can overuse CPU, but you cannot overuse RAM.

If running all your programs require more processing power than you have available your computer will just be slow.

On the other hand, if running all your program require more RAM than you have available, your computer will kill randomly processes that ask for too much RAM (at least on linux).

In Kubernetes it's a bit the same. You can ask for `limits` that are higher than the node CPU, but not the RAM.

Furthermore, you can ask for less than a CPU. But what does that mean? When you ask for 100 millicpu, Kubernetes will give you 100 milliseconds of CPU during a second of time. If your container wants to use more it'll be throttled during the remaining 900 milliseconds of the second.

## Good practices

Only the `requests` is taken into account into the scheduling. So it's possible for a Kubernetes node to go on ["overcommit"](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/resource-qos.md#qos-classes), to have the sum of resources used above the physical resource limitation of a node. In this case Kubernetes will look for pods to terminate. Its algorithm is to look at pods that are above what they requested and terminates them. If a pod has no `requests`, so they are using more than what they requested, they will be the first to be terminated. Other candidates are the pods that have gone over their request but are still under their limit.

Unless your applications are designed to use multiple cores, it is usually a best practice to keep the CPU request at "1" or below.

Most of the time you could define `request` == `limit`. But careful as your pod will be terminated if it goes above the `limit`.

## Exercices

First, you need to active the [`metric-server`](https://github.com/kubernetes-incubator/metrics-server/) on minikube:

```sh
minikube addons enable metrics-server
```

We'll use the [stress](https://people.seas.harvard.edu/~apw/stress/) tool to simulate resource utilization.
The used options will be:

* `--vm-bytes $MEM`: Amount of RAM used by stress, for example "150M" for 150 megabytes of RAM allocated.
* `--cpu $CPU`: Number of CPUs used by stress, an integer.

We will use other options, but ignore those. You can find more information on thos on the stress documentation.

### CPU

First let's try to declare a CPU `requests` that is too high compared to what your cluster has available.
Review and apply the file [01-cpu-requests.yml](./01-cpu-requests.yml). Look at the pod created. What can you see?

Second, let's try to have a pod consuming more CPU resources than allowed.
Review and apply the file [02-cpu-limits.yml](./02-cpu-limits.yml). Look at the pod created. What can you see? Look at the logs of the pods:

```sh
kubectl logs -f cpu-limits
```

Note, you can see the usage of the pod with kubectl:

```sh
kubectl top pod cpu-limits
```

### RAM

First let's try to declare a memory `requests` that is too high compared to what your cluster has available.
Review and apply the file [03-ram-requests.yml](./03-ram-requests.yml). Look at the pod created. What can you see?

Second, let's try to have a pod consuming more RAM resources than allowed.
Review and apply the file [04-ram-limits.yml](./04-ram-limits.yml). Look at the pod created. What can you see?

## Clean up

```sh
kubectl delete service,deployment,pod --all
```

## Links

* https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container
* https://medium.com/@betz.mark/understanding-resource-limits-in-kubernetes-cpu-time-9eff74d3161b
