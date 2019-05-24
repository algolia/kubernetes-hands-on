# Operators and CRDs: what, why, and how

## Introduction

In the previous section we saw what a controller is and what it does: interact with the Kubernetes API to act on it.

In fact, the Kubernetes API is more powerful. You can enhance it with your own objects. That's what is called Custom Resource Definition or CRD for short. When you add your own CRDs and your own controllers you get an operator.

We will not build on operator in this chapter, but we will rely on an existing operation, the [mysql-operator](https://github.com/oracle/mysql-operator).

Start by [deploying the operator](https://github.com/oracle/mysql-operator/blob/master/docs/tutorial.md). For ease of installation, the operator is included in this repo in the file [`01-mysql-operator.yml`](./01-mysql-operator.yml).

```sh
$ kubectl apply -f 20-operators/01-mysql-operator.yml
serviceaccount/mysql-operator created
serviceaccount/mysql-agent created
clusterrole.rbac.authorization.k8s.io/mysql-operator created
clusterrole.rbac.authorization.k8s.io/mysql-agent created
clusterrolebinding.rbac.authorization.k8s.io/mysql-operator created
clusterrolebinding.rbac.authorization.k8s.io/mysql-agent created
customresourcedefinition.apiextensions.k8s.io/mysqlclusters.mysql.oracle.com created
customresourcedefinition.apiextensions.k8s.io/mysqlbackups.mysql.oracle.com created
customresourcedefinition.apiextensions.k8s.io/mysqlrestores.mysql.oracle.com created
customresourcedefinition.apiextensions.k8s.io/mysqlbackupschedules.mysql.oracle.com created
deployment.apps/mysql-operator created
```

## CRDs

This operator creates multiple CRDs automatically, you can see it by:

```sh
$ kubectl describe crds mysqlclusters.mysql.oracle.com
Name:         mysqlclusters.mysql.oracle.com
[...]
API Version:  apiextensions.k8s.io/v1beta1
Kind:         CustomResourceDefinition
Metadata:
  [...]
Spec:
  [...]
  Group:       mysql.oracle.com
  Names:
    Kind:       Cluster
    List Kind:  ClusterList
    Plural:     mysqlclusters
    Singular:   mysqlcluster
  Scope:        Namespaced
  Version:      v1alpha1
  Versions:
    Name:     v1alpha1
    Served:   true
    Storage:  true
Status:
  [...]
Events:  [...]
```

We see that is created a new "kind" named `Cluster`, with a name `mysqlcluster`. And this new "kind" is completely integrated with Kubernetes and all the tooling try to do a:

```sh
kubectl get mysqlcluster
```

Magically it is also integrated with `kubectl` but only when connected to your cluster. So now we can write a [manifest](./02-mysql-cluster.yml) of kind `Cluster`:

```yml
apiVersion: mysql.oracle.com/v1alpha1
kind: Cluster
metadata:
  name: my-app-db
  namespace: default
spec:
  members: 1
```

We won't go into the details of what each option does. The only important part is that each of those options was declared in the CRD. Here we specify a mysql cluster with 1 node or `members`

## Operator

Now let's apply this manifest:

```sh
kubectl apply -f 20-operators/02-mysql-cluster.yml
```

And see what it did:

```sh
kubectl get mysqlcluster my-app-db
```

Look around in pods, deployments, services, stateful sets, secrets, etc. What can you see?

The operator creates a root password for mysql automatically, let's get it by

```sh
kubectl get secret my-app-db-root-password -o jsonpath="{.data.password}" | base64 --decode
```

Use it to connect to your mysql instance:

```sh
$ kubectl run mysql-client --image=mysql:5.7 -it --rm --restart=Never -- mysql -h my-app-db -uroot -p<PASSWORD> -e 'SELECT 1'
If you don't see a command prompt, try pressing enter.
Error attaching, falling back to logs: Internal error occurred: error attaching to container: container not running (ce62ea67ace6b65020d75646e3d31cad41f275ff51a05964a100ed8a87bf77c7)
mysql: [Warning] Using a password on the command line interface can be insecure.
+---+
| 1 |
+---+
| 1 |
+---+
pod "mysql-client" deleted
```

Change the number of `members` from `1` to `2` in the manifest of `02-mysql-cluster.yml`. Reapply it, what can you see? Try reconnecting to the mysql.

## Clean up

```sh
kubectl delete mysqlclusters,service,deployment,crds,pod --all
```

## Links
