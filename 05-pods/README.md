# The base building block: `pod`

## Introduction

In this section we will learn what is a pod, deploy your first container, configure k8s, and interact with k8s in the command line.

The base job of k8s is to schedule `pods`. K8s will choose how and where to schedule them. You can also see a `pod` as an object that requests some CPU and RAM. K8s will take those requirements into account in its scheduling.

But it has a base assumption that a `pod` can be killed whenever it wants to. So keep in mind that a `pod` is **mortal** and it **will** be destroyed at some point.

## First pod

Let's start to deploy this docker image <https://hub.docker.com/r/mhausenblas/simpleservice/>.
It's a stateless python JSON API that answers on:

* `/env`
* `/info`
* `/health`

Here is our first manifest for k8s:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-pod
spec:
  containers:
  - name: simple-pod
    image: mhausenblas/simpleservice:0.5.0
```

The manifest of k8s represents a desired state. We do not write the steps to go to this state. It's k8s who will handle it for us.
Let's have a look a the fields:

* `apiVersion`: the version of the k8s API we will be using, `v1` here
* `kind`: what resource this object represents
* `metadata`: some metadata about this `pod`, more on it later
* `spec`: specification of the desired behavior of this pod
  * `containers`: the list of containers to start in this pod
    * `name`: the name of the container
    * `image`: which image to start

Let's `apply` this manifest to k8s. This will tell k8s to create the `pod` and run it.

```bash
$ kubectl apply -f 05-pods/01-simple-pod.yml

pod "simple-pod" created
```

We also could have used the `kubectl create -f ...`. But it's better to have a declarative approach in k8s rather than an imperative one, [see]( https://medium.com/bitnami-perspectives/imperative-declarative-and-a-few-kubectl-tricks-9d6deabdde).

## `kubectl get`

Now list all the `pods` running in k8s. `get` is the `ls` of k8s.

```bash
$ kubectl get pod

NAME             READY     STATUS    RESTARTS   AGE
simple-pod   1/1       Running   0          4s
```

## Logs

Let's have a look at the logs of this `pod`:

```bash
$ kubectl logs simple-pod

2018-10-01T09:21:59 INFO This is simple service in version v0.5.0 listening on port 9876 [at line 142]
2018-10-01T09:23:21 INFO /info serving from 172.17.0.4:9876 has been invoked from 172.17.0.1 [at line 101]
2018-10-01T09:23:21 INFO 200 GET /info (172.17.0.1) 1.38ms [at line 1946]
```

## `kubectl describe`

Our first `pod` is now running. Now `describe` it. `describe` is a `get` on steroid, with more information.

```bash
$ kubectl describe pod simple-pod

[a lot of stuff]

IP: 172.17.0.1

[more stuff]
```

Look at the information provided. Get the field `IP`, it's the internal ip for this `pod`.

Connect to the cluster, and try to `curl` this ip - `172.17.0.4` in the example.

```bash
$ minikube ssh
$ curl 172.17.0.4:9876/info

{"host": "172.17.0.4:9876", "version": "0.5.0", "from": "172.17.0.1"}
```

K8s has a useful add-on, a web dashboard. It's included by default in minikube. You can start it with:

```bash
minikube dashboard
```

## Exercises

1. Deploy a `pod` containing nginx. The image name is `nginx`, see: <https://hub.docker.com/_/nginx/>
2. Do you think you can access the pod `simple-service` from outside of k8s, *without changing the manifest*?

## Clean up

```bash
kubectl delete pod --all
```

## Answers

For 2. Nop, the pod is only visible from the inside of the cluster

## Links

* https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/
* https://kubernetes.io/docs/concepts/workloads/pods/pod/
* https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
