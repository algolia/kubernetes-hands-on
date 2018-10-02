# Deploying my first application: `deployment`

## Introduction

In this section you will learn how to deploy a stateless application with multiple replicas and scale it.

Managing pods manually is doable, but what if you want to deploy multiple times the same one?
Of course you can copy/paste the yaml files and `apply` them. But remember, pods are **mortal**, so k8s can kill them whenever it feels like it.
So you can either have a look for your k8s cluster to recreate pods when needed, or you can use a `deployment`.

## First deployment

Let's create our first deployment:

```yml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: simple-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: simple-service
    spec:
      containers:
      - name: simple-service
        image: mhausenblas/simpleservice:0.4.0
        ports:
        - containerPort: 9876
```

Let's have a look a the configuration:
* `apiVersion`: Deployments are in "beta" (think google beta) in k8s so `apps/v1beta1`
* `kind`: A `deployment` has the kind `Deployment`
* `spec`:
  * `replicas`: The number of pods this `deployment` will create
  * `template`: The template of the pods to start
    * `metadata`: The metadata for this template, here only one label `app=simple-service`
    * `spec`: The `spec` of the pods
      * `containers`:
        * `ports`: The list of ports to expose internally in the k8s cluster
          * `containerPort`: The kind of port we want to expose, here a `containerPort`. So our container will expose one port `9876` in the cluster.

Apply the deployment:
```bash
$ kubectl apply -f 06-deployment/01-simple-deployment.yml
deployment.apps "simple-deployment" created
```

## Inside a `deployment`

Let's have a look at what this `deployment` created for us:

```bash
$ kubectl get deployment
NAME                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
simple-deployment   2         2         2            2           2m
```

Firstly, k8s created a `deployment`. We see a lot of 2s. It is the number of replicas that are available. Let's have a look at the pods we have running:

```bash
$ kubectl get pod

NAME                                 READY     STATUS        RESTARTS   AGE
simple-deployment-5f7c895db4-wpv6l   1/1       Running       0          1m
simple-deployment-5f7c895db4-wt9j7   1/1       Running       0          1m
```

The `deployment` created 2 pods for us, the number we put in `replicas`. We see that the pods have a unique name, but prefixed with the name of the deployment `simple-deployment`

Did k8s created something else for us? Let's have a look

```
$ kubectl get all

NAME                                     READY     STATUS    RESTARTS   AGE
pod/simple-deployment-5f7c895db4-wpv6l   1/1       Running   0          4m
pod/simple-deployment-5f7c895db4-wt9j7   1/1       Running   0          4m

NAME                                DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/simple-deployment   2         2         2            2           4m

NAME                                           DESIRED   CURRENT   READY     AGE
replicaset.apps/simple-deployment-5f7c895db4   2         2         2         4m
```

We see 3 things, you might have a section `ClusterIP` ignore it for now:
* `pod`: named `pod/[...]`
* `deployment`: named `deployment.apps/[...]`
* `replicaset`: named `replicaset.apps/[...]`

So in fact k8s created more `kind` than expected.
We won't go into details of what a [`ReplicaSet`](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) is, just keep it mind that it ensures that a specified number of pod replicas are running at any one time.

## Scale up

Let's play with our deployment now.
Update the number of `replicas` in the yaml, to a reasonnable number - say `5`.

```bash
$ kubectl apply -f 06-deployment/01-simple-deployment.yml
```

You can also use `kubectl scale`:
```bash
$ kubectl scale --replicas=5 -f 06-deployment/01-simple-deployment.yml
```

You can also edit the configuration in place with `kubectl edit`:
```bash
$ kubectl edit deployment simple-deployment
```

I would recommend to only use `kubectl apply` as it declarative and your local computer, so you can commit it afterwards.

What is happening? What changed?
You can use the flag `--watch` to `kubectl`, for example: `kubectl get pod --watch`.
Do not forget the `kubectl logs [...]` command.

## Scale down

Change again the number of replicas to `2`, reapply, see what is happening.

We know how to scale up/down a deployment, but how can we deploy a new version of the application. To achieve this, we need to tell k8s to update the image we are using in our deployment, for this:

```bash
$ kubectl set image deployment/simple-deployment simple-service=mhausenblas/simpleservice:0.5.0
deployment.apps "simple-deployment" image updated
```

Again, see what is happening.
Remember the command `kubectl describe deployment`.

## Exercices

1. Deploy multiple nginx. The image name is `nginx`, see: https://hub.docker.com/_/nginx/
2. Play with the scalling up/down & the deployment of new versions
3. Try accessing your deployment of nginx from outside of k8s, without changing the configuration. Do you manage to do it?
	* If yes, how did you manage to do it?
	* If no, what do you think is missing?

## Clean up

```bash
$ kubectl delete deployment,rs,pod --all
```