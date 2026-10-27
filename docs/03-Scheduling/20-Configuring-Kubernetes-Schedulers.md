# Configuring Kubernetes Schedulers
- Take me to [Video Tutorial](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/learn/lecture/14295628#overview)
  
In this section, we will take a look at configuring kubernetes schedulers.

![ks](../../images/ks.PNG)

How the scheduler works, using a simple example. There are 4 nodes.

- Node 01 - 4 CPU
- Node 02 - 4 CPU
- Node 03 - 12 CPU 
- Node 04 - 16 CPU


Let’s say we want to deploy the following pod with high priority.
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - image: simple-webapp-color
    name: simple-webapp-color
    resources:
      request:
        memory: "10Gi"
        cpu: 10
  priorityClassName: high-priority
```

Aside from this pod that needs to be deployed, there are others waiting to be deployed.

The scheduler carries out several filtering steps before determining which pods to deploy and where to deploy them.

**Scheduling Queue**

The Scheduling Queue is the first step, which filters pods based on their priority.

**Filtering**

The pod then enters this stage, which consists of filtering out nodes that are unable to provide the resource capacity required by the pod. Consequently, node 01 and node 02 are eliminated.

**Scoring**

Next comes the scoring phase, in which nodes are assigned a score based on the amount of available resources remaining after the pod is deployed on the node. In our example, node 04 achieves the highest score, with 6 CPUs remaining.

**Binding**

This is the final step: the pod is assigned to the node with the highest score—the one that best meets the pod's requirements.

All these sorting and filtering steps are performed using Kubernetes plugins, which are present in each of the four phases mentioned earlier; these plugins can be reused multiple times.

At each stage of the scheduler, there are "extension points." An extension point allows for the integration of these specific plugins.

It should be noted that there are extensions both before (`preFilter`) and after (`postFilter`) the main filtering stage—one of the four phases of the scheduler.

**Scheduling Queue**

<queueSort>
- PrioritySort

**Filtering**

<preFilter|filter|postFilter>
- NodeResourcesFit
- NodeName
- NodeUnschedulable
- NodeResourcesFit
- TaintToleration
- NodePorts
- NodeAffinity

**Scoring**

<preScore|score|reserve>
- NodeResourcesFit
- ImageLocality
- NodeResourcesFit
- TaintToleration
- NodeAffinity


**Binding**
<permit|preBind|bind|postBind>
- DefaultBinder

##### Configuring Kubernetes Schedulers Profiles

Scheduler profiles allow a single `kube-scheduler` binary to function as several distinct virtual schedulers.

In practical terms, this enables the application of entirely different placement rules—such as filters or scoring algorithms—based on the type of workload, all without needing to run multiple instances of the component.

Why use profiles instead of multiple binaries?
Before the introduction of profiles (in Kubernetes 1.18), if you required specific scheduling logic for certain pods, you had to deploy a second, custom `kube-scheduler` binary alongside the first. This created a major architectural issue: race conditions.

Because the two schedulers operated in isolation, they would occasionally attempt to assign pods to the same node resources simultaneously, without any coordination. With profiles, there is only one "brain" managing the global queue and the state of the cluster; this eliminates access conflicts while also reducing CPU and RAM consumption within the control plane.

To have multiple profiles within a scheduler, simply list them in the `profiles` section.
```
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: default-scheduler
- schedulerName: bin-packing-scheduler
- schedulerName: gpu-scheduler
```

Concrete Use Cases

- High Availability Profile (default-scheduler): The standard behavior. It aims to distribute the load evenly. If you deploy a ReplicaSet of 3 pods, it will prioritize placing them on 3 different nodes to ensure resilience.

- Big Data / Batch Profile (bin-packing-scheduler): A profile configured to do the opposite of the default. Instead of spreading out the load, it packs pods as tightly as possible onto the fewest number of nodes. The advantage? This leaves other nodes completely empty—ready to accommodate very large data-intensive pods that require a significant amount of RAM at once, or to be shut down by an auto-scaler to save costs.

- GPU-Specific Profile (gpu-scheduler): A profile featuring custom scoring plugins that examine hardware topology (e.g., NUMA nodes, NVLink connections between GPUs) to ensure optimal performance for artificial intelligence jobs.

To add a custom scoring plugin to a profile, everything happens in the component configuration file (usually called kube-scheduler.yaml or defined via a ConfigMap) which uses the KubeSchedulerConfiguration API.

Kubernetes uses the Scheduler Framework, which divides the pod placement lifecycle into several phases (extension points). To influence the sorting of the nodes, we intervene in the score phase.

```
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: default-scheduler

  - schedulerName: gpu-scheduler
    plugins:
      preScore:
        disabled:
          - name: "*"
      score:
        # 1. On peut désactiver certains plugins par défaut qui ne nous intéressent pas
        disabled:
          - name: ImageLocality 
        # 2. On active notre plugin personnalisé en lui donnant un poids (weight)
        enabled:
          - name: NodeResourcesFit
            weight: 1
          - name: MonSuperPluginGPU # Nom exact du plugin développé
            weight: 100 # Un poids très fort pour qu'il dicte le choix final
    
    # 3. (Optionnel) Passer des paramètres spécifiques à ton plugin
    pluginConfig:
      - name: MonSuperPluginGPU
        args:
          apiVersion: kubescheduler.config.k8s.io/v1
          kind: DefaultArgs # Ou une structure CRD spécifique à ton plugin
          favoriserNvidia: true
          toleranceChaleur: 85
```

How the score is calculated:
For each eligible node, the scheduler invokes all plugins in the score section. Each plugin returns a score from 0 to 100. The scheduler multiplies this score by the weight defined in your configuration, sums the results, and the node with the highest total is assigned the Pod.

## References
- https://github.com/kubernetes/community/blob/master/contributors/devel/sig-scheduling/scheduler.md
- https://kubernetes.io/blog/2017/03/advanced-scheduling-in-kubernetes/
- https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/
- https://stackoverflow.com/questions/28857993/how-does-kubernetes-scheduler-work


