### In-Place Resize of Pods

What is In-Place Resize of Pods?

Traditionally in Kubernetes, if you wanted to change the CPU or memory resources (requests or limits) of a running pod, you had to delete and recreate the pod.

In-Place Resize allows you to update a pod's CPU and memory resources while it is running, without terminating or recreating the pod itself (depending on the container runtime and the specified resizePolicy).

Configuration Example

Here is a practical example of a Pod configured with resizePolicy:

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-level-resize-demo
spec:
  containers:
  - name: pause
    image: registry.k8s.io/pause:3.9
    resizePolicy:
    - resourceName: cpu
      restartPolicy: NotRequired # Default: updates resources live without restarting
    - resourceName: memory
      restartPolicy: RestartContainer # Restarts the container to apply memory changes
    resources:
      requests:
        cpu: 100m
        memory: 100Mi

  - name: nginx-server
    image: registry.k8s.io/nginx:latest
    resizePolicy:
    - resourceName: cpu
      restartPolicy: RestartContainer
    - resourceName: memory
      restartPolicy: RestartContainer
    resources:
      requests:
        cpu: 200m
        memory: 200Mi
      limits:
        cpu: 200m
        memory: 200Mi
```

Key Concepts from the Example:

`resizePolicy`: Defines how the container should handle a resource update.

- `NotRequired`: The change is applied live to the container without restarting it (common for CPU).
- `RestartContainer`: The container is restarted so the new resource limits/requests take effect cleanly.

Pod-level resources: Resource requirements are defined at the container level (and can scale dynamically).

##### Limitations & Constraints

When using In-Place Pod Resize, keep the following rules in mind:

- Supported Resources: This feature applies only to CPU and memory.
- QoS Classes: Pod Quality of Service (QoS) classes cannot be changed dynamically.
- Container Types: Init containers and ephemeral containers are not supported.
- Removing Limits/Requests: Once resource requests or limits are set, they cannot be completely removed/unset.
- Memory Thresholds: A pod's memory cannot be resized below its current actual usage.
- OS Compatibility: Windows pods are not compatible with this feature (Linux nodes only).
