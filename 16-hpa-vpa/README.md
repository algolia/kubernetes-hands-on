# HPA, VPA

HPA means `Horizontal Pod Autoscaler`. It automatically scales the number of pods in a replication controller, deployment or replica set based on observed CPU utilization (or, with custom metrics support, on some other application-provided metrics).

Let's take an example with CPU usage:

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: simple-hpa
spec:
  maxReplicas: 10
  minReplicas: 3
  scaleTargetRef:
    apiVersion: extensions/v1beta1
    kind: Deployment
    name: simple-deployment
targetCPUUtilizationPercentage: 80
```

* `spec`: the spec for the HPA
  * `maxReplicas`: the maximum number of pods running
  * `minReplicas`: the minimum number of pods running
  * `scaleTargetRef`: what this HPA targets, here a `Deployment` named `simple-deployment`
* `targetCPUUtilizationPercentage`: the percentage of CPU utilization to cross to activate the HPA