# Naming things: `label` & `annotation`

In this section we will learn how to name things in k8s, and how to find them again.

Labels are the way to organise objects in kubernetes. The labels are a list of key/value.
Annotations are a way to mark objects in kubernetes, it's also a list of key/value.

They seem the same. The major difference is that you can query k8s based on labels.
On the other hand, annontations are not limited on characters used for labels.

Valid label keys have two segments: an optional prefix and name, separated by a slash `/`. The name segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character `[a-z0-9A-Z]` with dashes `-`, underscores `_`, dots `.`, and alphanumerics between. The prefix is optional. If specified, the prefix must be a DNS subdomain: a series of DNS labels separated by dots `.`, not longer than 253 characters in total, followed by a slash `/`.

Valid label values must be 63 characters or less and must be empty or begin and end with an alphanumeric character `[a-z0-9A-Z]` with dashes `-`, underscores `_`, dots `.`, and alphanumerics between.

Apply the pod `05-label-annotation/01-simple-service.yml`. It is the same as `04-pods/01-simple-service.yml` but with 2 labels:
* `env`: `production`
* `tier`: `backend`

Apply the pod `05-label-annotation/02-nginx.yml`. It is a simple nginx with 2 labels:
* `env`: `production`
* `tier`: `frontend`

Let's list all the pods that are in the `env=production`:

```bash
$ kubectl get pods -l env=production

NAME             READY     STATUS    RESTARTS   AGE
nginx            1/1       Running   0          13s
simple-service   1/1       Running   0          13s
```

Let's list all the pods that are in the `env=production,tier=frontend`:

```bash
$ kubectl get pods -l env=production,tier=frontend

NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          1m
```

Those queries we call them `selector` in the k8s jargon. We will use them later on with deployments.

## Exercices

Nothing to see here.

## Clean up

```bash
$ kubectl delete pod --all
```