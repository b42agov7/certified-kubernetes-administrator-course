### Vertical Pod Autoscaling (VPA)

Here the following Pod defintion file:
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
        requests:
          cpu: "250m"
        limits:
          cpu: "500m"
```

The pod will require 250m CPU and has a limit of 500 CPUs. This means that the maximum capacity is 500m CPU. If we have enabled the metric server, we can monitor the status of the Pod with the following command:

```
kubectl top pod my-app-pod
```

Let's say when the Pod reaches a power limit, we want to scale the Pod vertically. We can do this with the command `kubectl edit deployment my-app`

It works, but it will also kill the Pod and create a new one with the new values.

This is the manual method for performing VPA.

Let’s now look at how to avoid doing it manually.

##### Vertical Pod Autoscaling (VPA)

The VPA continuously monitors metrics; it can also increase or decrease the resources assigned to a pod within a deployment to balance the workload.

VPA does not have Kubernetes built-in; hence we must deploy it.

As a result, we must deploy it from the GitHub repository.

- https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler

Once it is deployed, we can see its components.

```
kubectl get pods --namespace=kube-system | grep vpa
```

We are going to meet there:
- vpa-admission-controller
- vpa-recommender
- vpa-updater

VPA Recommender:
The latter is responsible for monitoring resource usage via the Kubernetes API, collecting both historical and real-time usage data for pods, and subsequently providing recommendations for optimal CPU and memory values. These changes do not directly modify the Pod.

VPA Updater:
It detects Pods that are running with insufficient resources and evicts them when a resource update is required.
It receives information from the VPA Recommender and monitors the Pod. If a Pod needs to be updated, it evicts it. This means that it terminates the Pod.

VPA Admission COntroller
The admission controller intervenes during the pod creation process and leverages the recommender's suggestions to modify the pod specification, applying the recommended CPU and memory values ​​at startup. This ensures that newly created pods start with the correct resource requests.

Vertical Pod Autoscaling (VPA) creation with a YAML file:

There is no imperative commande to create it (it's not a built-in component).
```
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: "my-app"
        minAllowed:
          cpu: "250m"
        maxAllowed:
          cpu: "2"
        controlledResources: ["cpu"]
```

The `containerPolicies` define what is monitored, in this case, the CPU minimum allowed and max allowed are configured.

`updatePolicy`:  there are four modes that VPA operates in.
| Mode     | Description                                                                                                                                                                            |
|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Off      | Only recommends; does not change anything                                                                                                                                              |
| Initial  | Only changes on Pod creation; not later                                                                                                                                                |
| Recreate | Evicts pods if usage goes beyond range                                                                                                                                                 |
| Auto     | Updates existing pods to recommended numbers – For now, it   behaves like Recreate. But when support for "In-Place Update of Pod Resources" is  available, this mode will be preferred |

To check the recommendations:
```
kubectl describe vpa my-app-vpa
```

```
kubectl get vpa 
```

##### Key Differences

| Feature                 | VPA                                                            | HPA                                                              |
|-------------------------|----------------------------------------------------------------|------------------------------------------------------------------|
| Scaling Method          | Increases CPU and memory of existing Pods                      | Adds/Removes Pods based on load                                  |
| Pod Behavior            | Restarts Pods to apply new resource values                     | Keeps existing Pod running                                       |
| Handles Traffic Spikes? | No, because scaling requires a Pod restart                     | Yes, instantly adds more Pods                                    |
| Optimizes Costs?        | Yes, Prevents over-provisioning of CPU/memory                  | Avoids unnecessary idle Pods                                     |
| Best for                | Stateful workloads, CPU/memory-heavy apps (DBs, ML workloads)  | Web apps, microservices, stateless services                      |
| Example Use Cases       | Databases (MySQL, PostgreSQL), JVM-based apps, AI/ML workloads | Web servers (Nginx, API services), message queues, microservices |
