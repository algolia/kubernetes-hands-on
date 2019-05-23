# Running a stateful application: `volumes`

## Introduction

In this section you will learn how to deploy a stateful application, mysql in this example.

As you know a `pod` is mortal, meaning it can be destroyed by Kubernetes anytime, and with it it's local data, memory, etc. So it's perfect for stateless applications. Of course, in the real world we need a way to store our data, and we need this data to be persistent in time.

So how can we deploy a stateful application with a persistent storage in Kubernetes? Let's deploy a mysql.

## Volumes

We need to review what a volume is before continuing with the deployment of our mysql. As stated above, the disk of a pod is destroyed with it, so it's lost. For a database it'll nice if we could keep the data between restarts of the pods. Here comes the `volume`.

We can see a `pod` as something that requests CPU & RAM. We can see a `volume` as something that requests a storage on disk. Kubernetes handles a lot of different kind of volumes - 26 has this file hands on is written - from local disk storage to s3.

Here we will use `PersistentVolumeClaim`, it's an abstraction over the hard drives of the Kubernetes nodes - a fancy name for local hard drive.

Let's create the volume where our mysql data will be stored.

First we create the `PersistentVolume`. It is a piece of storage in the cluster that has been provisioned by a cluster administrator. It is a resource in the cluster just like a node is a resource of the cluster.

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Let's review some parameters:

* `capacity`: the capacity of the volume
  * `storage`: the volume size - here `1Gb`
* `accessModes`: how this volume will be accessed, here `ReadWriteOnce`
  * `ReadWriteOnce`: the volume can be mounted as read-write by a single node
  * `ReadOnlyMany`: the volume can be mounted read-only by many nodes
  * `ReadWriteMany`: the volume can be mounted as read-write by many nodes
* `hostPath`: where the storage will be stored on the host, here `/mnt/data`

Apply it:

```sh
kubectl apply -f 01-simple-mysql-pv.yml
```

Now that we have a storage, we need to claim it, make it available for our pods. So we need a `PersistentVolumeClaim`. It is a request for storage by a user. It is similar to a pod. Pods consume node resources and `PersistentVolumeClaim` consume `PersistentVolume` resources.

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

The manifest is pretty similar to the `PersistentVolume`:

```sh
kubectl apply -f 02-simple-mysql-pvc.yml
```

## Stateful application

Now let's create the `deployment` of mysql:

```sh
kubectl apply -f 03-simple-mysql-deployment.yml
```

There is a bunch of parameters we haven't seen yet:

* `strategy`: the strategy of updates of the pods
  * `type`: `Recreate`. This instructs Kubernetes to not use rolling updates. Rolling updates will not work, as you cannot have more than one Pod running at a time.
* `env`: the list of environment variables to pass to the container
  * `name`: the name of the env variable
  * `value`: the value of the env variable
* `volumeMounts`: the volumes, think directories, to give access to the container
  * `name`: the name of the volume to mount
  * `mountPath`: where in the container to mount the volume
* `volumes`: the volumes to request access to
  * `name`: the name of the volume, same as `volumeMounts.name`
  * `persistentVolumeClaim`: the `PersistentVolumeClaim` we want
    * `claimName`: the name of the claim

Let's finish by creating a `service` to have stable DNS entry inside our cluster.

```sh
kubectl apply -f 04-simple-mysql-service.yml
```

Finally let's access the mysql:

```sh
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword

If you don't see a command prompt, try pressing enter.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.02 sec)
```

Create a new database in mysql:

```sh
mysql> CREATE DATABASE testing;
Query OK, 1 row affected (0.01 sec)
```

Now delete the service and the deployment:

```sh
kubectl delete service,deployment --all
```

Recreate them, reconnect to mysql and see if you still have the database `testing` you created.

## Clean up

```sh
kubectl delete service,deployment,pvc,pv,pod --all
```
