# Pod distruption budget (PDB)

## Introduction

In Kubernetes pods are mortal and can be terminated at any time. When a pod is terminated it is called a [“disruption”](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/).

Disruptions can either be voluntary or involuntary. Involuntary means that it was not something anyone could expect (hardware failure for example). Voluntary means it was initiated by someone or something, like the upgrade of a node, a new deployment, etc.

Defining a “Pod Disruption Budget” helps Kubernetes manage your pods when a voluntary disruption happens. Kubernetes will try to ensure that not too many pods, matching a given selector, are unavailable at the same time

## PDB

A `PDB` is configured as this:

```yml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  minAvailable: 2 # or maxUnavailable
  selector:
    matchLabels:
      app: nginx
```

A PDB is composed of two configurations:

* the `selector` to know on which pods to apply this PDB
* a number either `minAvailable` or `maxUnavailable`. It can either be a fixed number like `2` in the example, or a percentage like `20%`

If you want to see the effect of a PDB, you will need a multi-node Kubernetes. As those lines are written `minikube` is a single node cluster. To have locally a multi-node cluster you can install [kind](https://github.com/kubernetes-sigs/kind).

Use the [configuration file](kind.yml) provided to create your cluster:

```sh
kind create cluster --config kind.yml
```

Review and apply the manifests in [01-pdb.yml](01-pdb.yml). Why did we specify a soft anti-affinity?

In a terminal run the command:

```sh
kubectl get pods -owide -w
```

It will display all the pods with the node where it's deployed. The `-w` is to watch the changes of those resources.

In another termimal run the command:

```sh
kubectl drain kind-worker2 --ignore-daemonsets
```

This command will remove, drain, the node `kind-worker2` from the cluster. Watch the output of this command and the changes in the `kubectl get pods -owide -w`.

What do you see? How can you explain this?

## Clean up

```sh
kubectl delete service,deployment,pod --all
```

## Links

* https://kubernetes.io/docs/tasks/run-application/configure-pdb/
