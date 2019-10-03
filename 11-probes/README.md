# Liveness and readiness probes, and how it impacts your pods

## Introduction

To be able to handle pods correctly, Kubernetes needs to know how they are behaving. For this it needs to have a way to know the state of the containers running inside a given pod. The creator of Kubernetes decided to go to a "black box" testing. "Black box" testing means that Kubernetes doesn't interact directly with the containers. It is each container that says how Kubernetes can know its state.

You can define two probes for Kubernetes to know the state of your container: "liveness" and "readiness" probes.

## Liveness probe

The liveness probe is here to detect if a container is still alive. Meaning, if the container is not in a broken state, in a dead lock, or anything related. This is always useful. It helps Kubernetes to know if your container is alive or not and so it can take decision based on that, like restarting it.

## Readiness probe

The readiness probe is here to detect if a container is ready to serve traffic. It is useful to configure when your container will receive external traffic sent by Kubernetes. Most of the time, when it's an API.

## Defining a probe

Both liveness and readiness probes have the same configuration. You have three ways of configuring a probe:

1. `exec` probe
1. `http` probe
1. `tcp` probe

### Exec probe

The `exec` probe lets you configure a command that Kubernetes will run in your container. If the command exits with a non zero status the probe will be considered unhealthy:

```yml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

The `exec` probe contains only a `command`, which is mandatory and represents which command it should run. Here it will be `cat /tmp/healthy`.
We will see later what `initialDelaySeconds` and `periodSeconds` means.

### HTTP probe

The `http` probe lets you configure a HTTP endpoint that Kubernetes will call in your container. If this endpoint returns a non 2XX status the probe will be considered unhealthy:

```yml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome
  initialDelaySeconds: 3
  periodSeconds: 3
```

The `http` probe has two mandatory fields `path` and `port` and one optional `httpHeaders`.

* `path`: lets you configure which http path the probe should call.
* `port`: lets you configure which port the probe should connect to.
* `httpHeaders`: lets you configure http headers the probe should send with its call.

### TCP probe

The `tcp` probe lets you configure a TCP port that Kubernetes will try to connect to. If it does not manage to establish a connection the probe will be considered unhealthy:

```yml
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
```

The `tcp` probe has one mandatory fields `port`. It represents which TCP port Kubernetes will try to connect to.

### `initialDelaySeconds` and `periodSeconds`

The `periodSeconds` field specifies that Kubernetes should perform the probe every `N` seconds. The `initialDelaySeconds` field tells Kubernetes that it should wait `N` second before performing the first probe.

If we take the example:

```yml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 5
```

This probe will wait 3 seconds before doing the first probing. The probing will be an http call to `http://localhost:8080/healthz`. After the first wait of 3 seconds, each probing will be done every 5 seconds.

## Impact of probes on Kubernetes

### Liveness probe impact

Look and apply the file [01-liveness-probe.yml](01-liveness-probe.yml).

Run `kubectl get pods -w` and see what is happening.

The `liveness` of this pod fails (`exit 1`), so Kubernetes detects that the pod is not alive anymore and restarts it. So this probe is key for Kubernetes to know if a pod should be restarted or not.

### Readiness probe impact

Look and apply the file [02-readiness-probe.yml](02-readiness-probe.yml).

Run `kubectl get pods -w` and see what is happening.
Run `kubectl get deployments -w` and see what is happening.

The pods are "Running" but not ready (the READY column to 0/1).

```sh
kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
readiness-deployment-5dd7f6ff87-jsm9f   0/1     Running   0          2m17s
readiness-deployment-5dd7f6ff87-wnrmg   0/1     Running   0          2m17s
```

If you try to access the service `readiness-service`, with `kubectl port-forward service/readiness-service 8000:80`. It won't work. Kubernetes sees that all the pods are not ready, so it won't send traffic to them.

The readiness probe is also used when you do rolling updates. Kubernetes will wait for the pods with the new version to be ready before sending traffic to it.

### Good practices

Most of time having the readiness and liveness probe to be the same is enough. In some cases you might want to them to be different. A good example is a container running a mono-threaded application that accept HTTP calls (who said PHP). Let’s say you have an incoming request that will very long to be processed. Your application is not able to receive any other request, as it’s blocked by the incoming requests. So it’s not “ready” on the other hand it’s processing a request so it’s “alive”.

Another tip, your probes should not call dependent services of your application, to prevent cascading failure.

## Clean up

```sh
kubectl delete service,deployment,pod --all
```

## Links

* https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/
