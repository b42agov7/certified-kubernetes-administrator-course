# Security Context
  - Take me to [Video Tutorial](https://kodekloud.com/topic/security-contexts-2/)
  
In this section, we will take a look at security context

## Container Security
 ```
 $ docker run --user=1001 ubuntu sleep 3600
 $ docker run -cap-add MAC_ADMIN ubuntu
 ```
 
 ![csec](../../images/csec.PNG)
 
## Kubernetes Security
- You may choose to configure the security settings at a container level or at a pod level.

 ![ksec](../../images/ksec.PNG)

## Security Context
- To add security context on the container and a field called **`securityContext`** under the spec section.
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    securityContext:
      runAsUser: 1000
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
  ```
  ![sxc1](../../images/sxc1.PNG)
  
- To set the same context at the container level, then move the whole section under container section.
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
  ```
  ![sxc2](../../images/sxc2.PNG)
  
- To add capabilities use the **`capabilities`** option
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: web-pod
  spec:
    containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities: 
          add: ["MAC_ADMIN"]
  ```
  ![cap](../../images/cap.PNG)
  
  
### K8s Reference Docs
- https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

As we saw in the previous lecture, when you run a Docker container, you have the option to define

a set of security standards, such as the ID of the user used to run the container, the Linux capabilities

that can be added or removed from the container, etc. these can be configured in Kubernetes as well

```
$ docker run --user=1001 ubuntu sleep 3600
```

```bash
$ docker run -cap-add MAC_ADMIN ubuntu
```

as you know already in Kubernetes.

Containers are encapsulated in pods.

You may choose to configure the security settings at a container level or at a pod level.

If you configure it at a pod level, the settings will carry over to all the containers within the pod.

If you configure it at both the pod and the container, the settings on the container will override

the settings on the pod.

Let us start with a pod definition file.

This pod runs an ubuntu image with the sleep command to configure security context on the container.

Add a field called Security Context under the specs section of the pod.

Use the run as user option to set the user ID for the pod.

```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
```

To set the same configuration on the container level.

Move the whole section under the container specification like this.

```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
```

To add capabilities.

Use the capabilities option and specify a list of capabilities to add to the pod.

```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```

*Note: Capabilities are only supported at the container level and not at the POD level.*
