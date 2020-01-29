# Stateful Sets

## Introduction

Like a `Deployment`, a `StatefulSet` manages Pods that are based on an identical container spec. Unlike a `Deployment`, a `StatefulSet` maintains a sticky identity for each the pods. These are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

`StatefulSets` are valuable for applications that require one or more of the following.

* Stable, unique network identifiers, ex: distributed system, like ElasticSearch
* Stable, persistent storage, ex: MySQL
* Ordered, graceful deployment and scaling
* Ordered, automated rolling updates, ex: MySQL Master+Slave

```yml
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

As you can see the manifest is very close to the one of a deployment. Apply the manifest [01-statefulset.yml](01-statefulset.yml).

Look at the pods generated, see how they are generated. Connect to one of the pods:

```sh
kubectl exec -ti web-0 /bin/bash
```

Write a file in the volume `www`. Terminate the same pod. See what happens. Reconnect to the pod, look at volume `www`. What can you see?

## Clean up

```sh
kubectl delete statefulset,deployment,service,pod --all
```
