# The base building block: `pod`

## Introduction

In this section we will learn what is a pod, deploy your first container, configure k8s, and interact with k8s in the command line.

The base job of k8s is to schedule pods. K8s will choose how and where to schedule them. You can also see a pod as an object that requests some CPU & RAM, and k8s will take those requirements into account in his scheduling.

But it has a base assumption that a pod can be killed whenever it wants to. So keep in mind that a pod is **mortal** and it **will** be destroyed at some point.

## First pod

Let's start to deploy this docker image https://hub.docker.com/r/mhausenblas/simpleservice/.
It's a stateless python JSON API that answers on:

* `/env`
* `/info`
* `/health`

Here is our first YML configuration for k8s:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-service
spec:
  containers:
  - name: simple-service
    image: mhausenblas/simpleservice:0.5.0
```

The YML configuration of k8s represents a desired state. We do not write the steps to go to this state. It's k8s who will handle it for us.
Let's have a look a the fields:

* `apiVersion`: the version of the k8s API we will be using, `v1` here
* `kind`: what resource this object represents
* `metadata`: some metadata about this pod, more on it later
* `spec`: specification of the desired behavior of this pod
  * `containers`: the list of containers to start in this pod
    * `name`: the name of the this container
    * `image`: which image to start

Let's `apply` this configuration k8s. This will tell k8s to create the pod and run it.

```bash
$ kubectl apply -f 04-pods/01-simple-service.yml

pod "simple-service" created
```

We also could have used the `kubectl create -f ...`. But it's better to have a declarative approach in k8s rather than an imperative one (see: https://medium.com/bitnami-perspectives/imperative-declarative-and-a-few-kubectl-tricks-9d6deabdde)

## `kubectl get`

Now list all the pods running in k8s. `get` is the `ls` of k8s.

```bash
$ kubectl get pod

NAME             READY     STATUS    RESTARTS   AGE
simple-service   1/1       Running   0          4s
```

## Logs

Let's have a look at the logs of this pod:

```bash
$ kubectl logs simple-service

2018-10-01T09:21:59 INFO This is simple service in version v0.5.0 listening on port 9876 [at line 142]
2018-10-01T09:23:21 INFO /info serving from 172.17.0.4:9876 has been invoked from 172.17.0.1 [at line 101]
2018-10-01T09:23:21 INFO 200 GET /info (172.17.0.1) 1.38ms [at line 1946]
```

## `kubectl describe`

Our first pod is now running. Now `describe` it. `describe` is a `get` in steroid, with more information.

```bash
$ kubectl describe pod simple-service

[a lot of stuff]

IP: 172.17.0.1

[more stuff]
```

Look at the information provided. Get the field `IP`, it's the internal ip for this pod

Connect to the cluster, and try to `curl` this ip - `172.17.0.4` in the example.

```bash
$ minikube ssh
$ curl 172.17.0.4:9876/info

{"host": "172.17.0.4:9876", "version": "0.5.0", "from": "172.17.0.1"}
```

K8s has a useful add-on, a web dashboard. It's included by default in minikube. You can start it with:

```bash
$ minikube dashboard
```

## Exercises

1. Deploy a pod containing nginx. The image name is `nginx`, see: https://hub.docker.com/_/nginx/
2. Try accessing the pod `simple-service` from outside of k8s, without changing the configuration. Do you manage to do it?
	* If yes, how did you manage to do it?
	* If no, what do you think is missing?

## Clean up

```bash
$ kubectl delete pod --all
```