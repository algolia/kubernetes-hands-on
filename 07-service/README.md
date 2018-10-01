# Accessing my first application: `service`

In this section you will learn how to access your application from outside your cluster, and do service discovery.

If it's not already done install the `minikube` addon `coredns` & `ingress`
```bash
$ minikube addons enable coredns
coredns was successfully enabled

$ minikube addons enable ingress
ingress was successfully enabled
```

You are able to deploy an image with multiple replicas, but it is not very convenient to access it. You need to know the IP of a `pod` to be able to target your application. And it's not accessible from the outside of the cluster.

What we need is a `service`. It'll allow us to acces our pods internally or externally.

First apply the deployment:
```bash
$ kubectl apply -f 07-service/01-simple-deployment.yml
deployment.apps "simple-deployment" created
```

Now start another container. We will use it to see what we can access internaly inside k8s:

Apply the pod:
```bash
$ kubectl apply -f 07-service/02-bash.yml
pod "bash" created
```

And connect to it
```bash
$ kubectl exec -it bash -- /bin/bash
root@bash:/#
```

Install `dnsutils` & `curl` in the container, you will need them:
```bash
root@bash:/# apt update && apt install dnsutils curl
[...]
```

You now have a shell inside a k8s pod running in your cluster. Let this console open so you can type commands.
Try to curl one of the pods created by the deployment above. How can you access the `deployment` **without** targeting a specific `pod`?

Ok, now let's create our first service:

```yml
apiVersion: v1
kind: Service
metadata:
  name: simple-internal-service
spec:
  ports:
  - port: 1234
    targetPort: 9876
  selector:
    app: simple-service
```

Let's have a look a the configuration:
* `kind`: A `service` has the kind `Service`
* `spec`:
  * `ports`: the list of ports to expose. Here we export `port` `1234`, but redirect internally all traffic to the `targetPort` `9876`
  * `selector`: **TODO**

Apply the service:
```bash
$ kubectl apply -f 07-service/03-internal-service.yml
service "simple-internal-service" created
```

Your service is now accessible internaly, try this in your `bash` container
```bash
root@bash:/# nslookup simple-internal-service
Server:		10.96.0.10
Address:	10.96.0.10#53

Name:	simple-internal-service.default.svc.cluster.local
Address: 10.96.31.244
```

Try to curl the `/info` url, remember the `ports` we choose in the `service`.

Can you access this service from the outside of k8s?

The answer is no, it's not possible. To do this you need an `ingress`. Ingress means "entering into".

You need to connect internet to the ingress that'll connect it to a service:

```
 internet
     |
[ ingress ]
--|-----|--
[ service ]
```

Let's create our ingress:

```yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: simple-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  backend:
    serviceName: simple-internal-service
    servicePort: 80
```

Let's have a look at the configuration:

* `kind`: Ingress
* `metadata`:
  * `annotations`: some annotations specific to the ingress
    * `nginx.ingress.kubernetes.io/ssl-redirect`: To fix a redirect, see [this](https://github.com/kubernetes/ingress-nginx/issues/1567). This is only used if you use the nginx ingress
* `spec`:
  * `backend`: the default backend to redirect all the requests to
    * `serviceName`: the name of the k8s traffic to redirect to
    * `servicePort`: the port of the service

Apply the ingress:
```bash
$ kubectl apply -f 07-service/04-ingress.yml
ingress.extensions "simple-ingress" created
```

Get the IP of your minikube cluster:
```bash
$ minikube ip
192.168.99.100
```

Open a browser on `http://192.168.99.100/info`

With this configuration we have a `deployment` that manages pods. A `service` that gives access to the pods, and an `ingress` that gives access to the pod to the external world.

## Exercices

1. Deploy an nginx and expose it internally
2. Read [this](https://kubernetes.io/docs/concepts/services-networking/ingress/#simple-fanout) and modify the ingress to have:
  * `/simple` that goes to the `simple-service`
  * `/nginx` that goes to your nginx deployment

## Clean up

```bash
$ kubectl delete ingress,service,deployment,rs,pod --all
```