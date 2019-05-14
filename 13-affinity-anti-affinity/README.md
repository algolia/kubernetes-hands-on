# Affinity and anti-affinity

## Introduction

When you declare a `pod` you can declare where you would like it to run. Either by forcing pods to run together on the same node, or forcing pods not to run on the same node.

The first two are pod affinity/anti-affinity, the last one is node affinity.

## Pod affinity/anti-affinity

Disclaimer: The `affinity` feature is very powerful, and we will only have a look at a part of it, the inter-pod (anti-)affinity.

Pod affinity and anti-affinity are declared at the pod level and hints k8s scheduler to run (or not run) some pods on the same node.

Running the same pods on the same host can improve network performances. A use case would be to try to have your application and its cache on the same node, to reduce latency.

Hinting k8s to run multiple pods on different nodes is a good way to improve fail-over. Example, if you have 3 nodes, and an application replicated 3 times. It would be unwise to have all the pods running on the same node. With pod anti-affinity you can ask k8s to schedule one pod on each node.

### Common configuration

The `affinity` is specified at the pod level. The configuration for `podAffinity` or `podAntiAffinity` is the same. So let's look at a `podAntiAffinity`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    run: nginx
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: run
                operator: In
                values:
                  - nginx
          topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx
```

* `podAntiAffinity`: declares it will be an anti-affinity. Use `podAffinity` for pod affinity.
  * `requiredDuringSchedulingIgnoredDuringExecution`: declares when to apply this `affinity`. Here we require this affinity to be applied at scheduling time, but ignore it at runtime. If the labels are changed at runtime, this affinity won't be recomputed.
    * `labelSelector`: on which selector the affinity should work on, here we will select based on labels
      * `matchExpressions`: how we will match on the labels
        * `key`: which label name we will use for this match expression, here `run`
        * `operator`: how we should match the value of the label, here `In`, meaning equal
        * `values`: which label names we will match on
    * `topologyKey`: which label of the node the affinity should use

In english words, this configuration means that we want to ensure that pods with the label `run=nginx` will not run on node with the same hostname (`kubernetes.io/hostname`).

You also have `preferredDuringSchedulingIgnoredDuringExecution` to not require but only hints the scheduler. Carefull the configuration for this is different:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pdb
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          podAffinityTerm:
            labelSelector:
              matchExpressions:
                - key: run
                  operator: In
                  values:
                    - nginx
            topologyKey: kubernetes.io/hostname
  containers:
    - name: nginx
      image: nginx
```

You need to specify a `weight` and a `podAffinityTerm`.

For a complete overview of all the options, have a look at the [specs](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/podaffinity.md).

### `affinity`

It is difficult to test the `affinity` with minikube as there is only one node to the cluster. We won't be able to see it in action. So we will not demonstrate this feature.

### `anti-affinity`

Review and apply the manifests in [01-pod-anti-affinity.yml](01-pod-anti-affinity.yml).

Describe the pods what can you see? How do you explain it?

Delete the pods, and try to use `preferredDuringSchedulingIgnoredDuringExecution` instead of `requiredDuringSchedulingIgnoredDuringExecution`. Reapply the manifest. What can you see? How do you explain it?

## Node affinity

Node affinity is very close to pod affinity. Instead of specifying a `podAffinity` you define a `nodeAffinity`. As above, complete overview of all the options, have a look at the [specs](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/nodeaffinity.md).

Each resource in k8s can have labels, even nodes. You can see them with `kubectl`:

```bash
kubectl get nodes --show-labels
```

You can add labels to a node with:

```bash
kubectl label nodes [nodeName] gpu=yes
```

Here we add the label `gpu` with value `yes` to the node `node1`.

You can force a pod to be scheduled on a specific node by adding right `nodeAffinity` to the spec of it. Here we would like to pod to be scheduled on a node with the label `gpu=yes`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: gpu
              operator: In
              values:
                - "yes"
  containers:
  - name: nginx
    image: nginx
```

Review and apply the manifests in [02-node-affinity.yml](02-node-affinity.yml).

Describe the pods what can you see? How do you explain it?

## Exercices

Nothing to see here.

## Clean up

```bash
kubectl delete service,deployment,pod --all
```

## Links

* https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
