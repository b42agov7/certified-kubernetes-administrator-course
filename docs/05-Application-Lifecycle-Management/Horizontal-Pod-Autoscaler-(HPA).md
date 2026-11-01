###  Horizontal Pod Autoscaler (HPA)

##### Manually Scaling a Workload

How we would go about manually scaling a workload?

Let’s say we have a deployment, and we need to ensure that the workload for our deployment’s application remains consistent.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: nginx
          resources:
            request:
              cpu: "250m"
            limits:
              cpu: "500m"
```

We can see that the limit for our pod is "500m" in terms of CPU. If we have deployed the `Metrics Server` in our cluster, we can use the command `kubectl top pod my-app-pod`.

If the command output shows CPU consumption close to the limit, we can scale up using the command `kubectl scale deployment my-app --replicas=3`.

This is not an appropriate method, as it requires constant manual monitoring of the pods.

To effectively address our core problem, we can use the Horizontal Pod Autoscaler (HPA).

##### Horizontal Pod Autoscaler (HPA)

The Horizontal Pod Autoscaler (HPA):

- Continuously monitors metrics.
- Increases or decreases the number of pods as needed, based on CPU or RAM metrics.
- Can manage thresholds.
- Can also track multiple types of metrics.

To create a Horizontal Pod Autoscaler (HPA) manually, we use the following command:

```
kubectl autoscale deployment my-app \
     --cpu-percent=50 --min=1 --max=10
```

This command sets a CPU threshold of 50%, with a minimum of 1 pod and a maximum of 10 pods.

When this command is used, it creates an Horizontal Pod Autoscaler named "my-app" for our deployment. 

This autoscaler reads the limits configured within the pod and continuously queries the Metrics Server to monitor the pods' resource consumption. 

If usage exceeds 50%, it increases the number of pods; conversely, it can subsequently decrease the number of pods.

To list our HPA use:
```
kubectl get hpa
```

`TARGETS` colums informs us of the current value that will trigger our threshold.

When we don't need anymore our HPA, we use this kubectl command:
```
kubectl delete hpa my-app
```

Be aware that a declarative approach exists.

##### Declarative approach

We need to create an HPA definition file.

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

HPA comes built-in with Kubernetes since version 1.23

Until now, the HPA has relied on the internal Metrics Server; however, it can also draw from other sources thanks to a Custom Metrics Adapter which can retrieve data from various internal origins, including the workload deployed within a cluster.

Furthermore, the HPA can be configured to utilize external sources through an External Adapter, such as Datadog or Dynatrace.

