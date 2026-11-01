### Autoscaling
What does scaling mean?

Let’s go back to the time when applications were hosted on servers. What happens if the load on an application increases and there are no resources left? We would have to increase the server's physical resources—which first required shutting the server down. This is known as vertical scaling.

If an application can run as multiple instances, we can use horizontal scaling. This involves adding more servers (or instances) and distributing the load among them.

In short, horizontal scaling consists of adding more instances/resources (scale out).

Horizontal autoscaling consists of automatically adding more instances or servers for our application based on demand.

In Kubernetes, there are two main targets for scaling:

- Scaling Workload, which consists of adding or removing pods.
- Scaling Cluster Infrastructure, which consists of adding or removing nodes (servers) in our cluster.

In both cases, we can apply either **horizontal** or **vertical** methods:

For Scaling Cluster Infrastructure: Horizontal scaling involves adding more nodes, while vertical scaling involves increasing the CPU/RAM of the existing nodes.
For Scaling Workload: Horizontal scaling involves adding more pods, while vertical scaling involves increasing the CPU/RAM resources of the existing pods.

Furthermore, scaling can be performed in two ways: **manual** or **automated**.

##### Manual Approach

Cluster Infrastructure Scaling

- Horizontal Scaling: The manual approach is to add additional worker nodes to the cluster (for example, by provisioning a new server and running the kubeadm join command).
- Vertical Scaling: Manually resizing cloud instances (VMs) usually requires restarting the server, though some modern cloud providers allow hot-plugging. In Kubernetes itself, changing node sizes is managed at the cloud provider/infrastructure level rather than via kubectl.

Workload Scaling

- Horizontal Scaling: The manual approach is to use the kubectl scale command (or update the replica count in a Deployment manifest).
- Vertical Scaling: The manual approach involves editing the pod or deployment specification (kubectl edit) to adjust the requests and limits of CPU and memory. (Note: Depending on the Kubernetes version and object type, modifying a pod's resources directly might require recreating the pod).

##### Automated Approach

The automated method relies on Kubernetes controllers:

- Horizontal Scaling for Cluster Infrastructure: Handled by the Cluster Autoscaler, which automatically adds or removes nodes based on pending pods or underutilized nodes.
- Horizontal Scaling for Workload: Handled by the Horizontal Pod Autoscaler (HPA), which adjusts the number of replica pods based on CPU utilization or other metrics.
- Vertical Scaling for Workload: Handled by the Vertical Pod Autoscaler (VPA), which automatically adjusts the CPU and memory reservations for your pods.
