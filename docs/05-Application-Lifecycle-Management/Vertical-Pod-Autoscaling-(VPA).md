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



##### Key Differences

| Feature                 | VPA                                                            | HPA                                                              |
|-------------------------|----------------------------------------------------------------|------------------------------------------------------------------|
| Scaling Method          | Increases CPU and memory of existing Pods                      | Adds/Removes Pods based on load                                  |
| Pod Behavior            | Restarts Pods to apply new resource values                     | Keeps existing Pod running                                       |
| Handles Traffic Spikes? | No, because scaling requires a Pod restart                     | Yes, instantly adds more Pods                                    |
| Optimizes Costs?        | Yes, Prevents over-provisioning of CPU/memory                  | Avoids unnecessary idle Pods                                     |
| Best for                | Stateful workloads, CPU/memory-heavy apps (DBs, ML workloads)  | Web apps, microservices, stateless services                      |
| Example Use Cases       | Databases (MySQL, PostgreSQL), JVM-based apps, AI/ML workloads | Web servers (Nginx, API services), message queues, microservices |
