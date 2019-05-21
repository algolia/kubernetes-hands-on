# HPA, VPA

## HPA

HPA means `Horizontal Pod Autoscaler`. It automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization (or, with custom metrics support, on some other application-provided metrics).

Let's take an example with CPU usage:

```yml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: simple-hpa
spec:
  maxReplicas: 10
  minReplicas: 3
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-deployment
targetCPUUtilizationPercentage: 80
```

* `spec`: the spec for the HPA
  * `maxReplicas`: the maximum number of pods running
  * `minReplicas`: the minimum number of pods running
  * `scaleTargetRef`: what this HPA targets, here a `Deployment` named `simple-deployment`
* `targetCPUUtilizationPercentage`: the percentage of CPU utilization to cross to activate the HPA

Let's see an HPA in action. This exemple is inspired by [the official Kubernetes documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/).

To see it in action, we need to have a workload that consumes some CPU. We'll use an image that is CPU intensive: https://hub.docker.com/r/googlecontainer/hpa-example.

This image computes the square root of numbers:

```php
<?php
  $x = 0.0001;
  for ($i = 0; $i <= 1000000; $i++) {
    $x += sqrt($x);
  }
  echo "OK!";
?>
```

First, you need to active the [metric-server](https://github.com/kubernetes-incubator/metrics-server/) on minikube:

```sh
minikube addons enable metrics-server
```

Review and apply the file [01-hpa.yml](01-hpa.yml).

Now let's generate some load on our service:

```sh
kubectl run -i --tty load-generator --image=busybox /bin/sh
```

Then run in this terminal:

```sh
while true; do wget -q -O- http://hpa-example; done
```

In another terminal look at the state of the pods and the hpa, remember the `-w` flag on `kubectl`.

Wait a bit and see how it changes the pods and the hpa.

Stop the load generator, again, wait a bit and see how it changes the pods and the hpa.

## VPA

VPA means `Vertical Pod Autoscaler`. It automatically find the right resources for pods in a replication controller, deployment or replica set based on observed CPU utilization.

This feature is in beta, understand you can install it in your cluster but it's not integrate with the standard Kubernetes source code.

Testing the VPA requires some work. You need to have "metric-server" installed:

```sh
minikube addons enable metrics-server
```

1. Clone the [autoscaler](https://github.com/kubernetes/autoscaler) reposority:

```sh
git clone git@github.com:kubernetes/autoscaler.git
```

1. Install the [vpa](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler):

```sh
cd vertical-pod-autoscaler && ./hack/vpa-up.sh
```

1. Try the exemple from the [vpa github repo](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler#test-your-installation):

```sh
kubectl apply -f examples/hamster.yaml
```

Look at the file [hamster.yaml](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler/examples). It contains the VPA definition:

```yml
apiVersion: "autoscaling.k8s.io/v1beta2"
kind: VerticalPodAutoscaler
metadata:
  name: hamster-vpa
spec:
  targetRef:
    apiVersion: "extensions/v1beta1"
    kind: Deployment
    name: hamster
```

The only interesting part is the `targetRef`. It's only which Kubernetes object the VPA will act on. Here the `Deployment` named `hamster`.

The second part is a regular deployment, that have undersized CPU `requests`.

After applying those manifest, look at the resources requests for the deployment.

## Clean up

```sh
kubectl delete service,deployment,pod,crd,vpa --all
```

## Links

* https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
