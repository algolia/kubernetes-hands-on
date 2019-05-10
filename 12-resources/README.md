# Resources, and how it impacts the scheduling

## Introduction

When you declare a `pod` you can declare what resources it will use. Those resources are how much CPU and RAM your pods will use.

Specifying those will help k8s to schedule more efficently your pods on the available nodes.

For each resource you can define the `limits` and the `requests`.

## Resources definition

The CPU resource is measured in a number of CPU the pod will use for a given amount of time. It can be inferior to 0.
Specifying `0.5` (or `500m`, which means 500 millicpu), will give half of a CPU to your pod.

The RAM resource is measured in the number of bytes of RAM the pod will use.
Specifying `123Mi` (or `128974848`, which means 128974848 bytes), will give that many RAM to your pod.

## `requests` vs `limits`

k8s let's you configure the `requests` and the `limits` for each resource.

### `requests`

The `requests` is the number that will help k8s schedule your pod on a node were the resources are available.

Let's take an example. You have 3 nodes:

* node 1 has `100m` CPU available, and `1Gi` RAM available
* node 2 has `1000m` CPU available, and `100Mi` RAM available

1. You start a pod with only a CPU request of `10m`. It can be scheduled on any node, so k8s will take either of them.
1. You start a pod with only a CPU request of `500m`. It can be scheduled only on node 2. Node 1 has only `100m` CPU available.
1. You start a pod with a CPU request of `500m` and a RAM request of `500Mi`. k8s will not be able to schedule your pod as no node has both requests at the same time. The pod will be in the state `Unschedulable`.

### `limits`

The `limits` is the maximum utilization of a resource k8s will allow for a given pod. If your pod goes above this limit it will be restarted.

## Good practices

Only the `requests` is taken into account into the scheduling. So it's possible for a k8s node to go on ["overcommit"](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/node/resource-qos.md#qos-classes), to have the sum of resources used above the physical resource limitation of a node. In this case k8s will look for pods to terminate. Its algorithm is to look at pods that are above what they requested and terminates them. If a pod has no `requests`, so they are using more than what they requested, they will be the first to be terminated. Other candidates are the pods that have gone over their request but are still under their limit.

Unless your applications are designed to use multiple cores, it is usually a best practice to keep the CPU request at "1" or below.

Most of the time you could define `request` == `limit`. But careful as your pod will be terminated if it goes above the `limit`.

## Exercices

Nothing to see here.

## Clean up

```bash
kubectl delete service,deployment,pod --all
```

## Links

* https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container
