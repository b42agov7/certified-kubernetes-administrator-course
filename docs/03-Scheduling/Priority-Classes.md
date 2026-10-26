### Priority Classes

  - Take me to [Video Tutorial](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/learn/lecture/50065393#overview)

In this section, we will take a look at Priority Classes

##### Priotities

As we know, Kubernetes runs applications in the form of Pods. Some of these applications may be mission-critical, while others are less so. It is essential to ensure that our critical applications remain operational at all times and are not disrupted by less critical ones.

To do this, we must use Priority Classes. Priority Classes ensure that critical applications can run ahead of less critical ones.

If a critical application cannot run—or be scheduled—due to a less critical application, the scheduler will terminate that less critical application.

To do this, we must use Priority Classes. Priority Classes ensure that critical applications can run ahead of less critical ones.

If a critical application cannot run—or be scheduled—due to a less critical application, the scheduler will terminate that less critical application.

Priority Classes are not created within a specific namespace. They are therefore available to all pods in any namespace.

##### Range priorities.

Priority Classes are defined using a numerical range, spanning from 1,000,000,000 down to -2,147,483,648.

A high number indicates high priority, while a low—or even negative—number indicates low priority.

This range applies to our Pods—specifically, application Pods, or "workloads."

There is a separate, specialized range for critical internal Kubernetes components (the control plane), which can extend up to 2,000,000,000.

**Listing Priority Classes**

Use this kubectl command:
```
kubectl get priorityClass
```

```
kubectl get priorityClass
NAME                        VALUE        GLOBAL-DEFAULT   AGE    PREEMPTIONPOLICY
system-cluster-critical     2000000000   false            461d   PreemptLowerPriority
system-node-critical        2000001000   false            461d   PreemptLowerPriority
```

**Describe Priority Classes**

Use this kubectl command:
```
kubectl describe priorityClass <Priority Classes Name>
```

**Get a quick Priority Class YAML template**
```
kubectl get priorityClass system-node-critical -o yaml
```

##### Create a Priority Class

```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```

##### Associating a Priority Class with a pod

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
      - containerPort: 8080
  PriorityClassName: high-priority
```

In the Pod definition file, specify the name of the PriorityClass in the `PriorityClassName:` field of the Pod.

If we do not assign a PriorityClass to a Pod, it will default to 0 (to all pods).

If you want to change this behavior, you must create a new PriorityClass and set the `globalDefault` field to true (only one).

##### Effect of Pod Priotity

If a cluster already has applications deployed, and a high-priority application cannot be deployed due to the existing applications, what should be done? 
We must use one of the `preemptionPolicy` effects within the definition file for a priority class.

```
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
preemptionPolicy: PreemptLowerPriority
```


If the `preemptionPolicy` field is not defined, it defaults to `PreemptLowerPriority`.

This means it will kill jobs with a lower priority to allow higher-priority pods to run.

If we do not want to kill jobs with a lower priority, but rather wait until resources become free again, we must set the value of the `preemptionPolicy` field to `never`.
