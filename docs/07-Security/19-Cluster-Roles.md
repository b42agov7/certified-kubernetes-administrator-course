# Cluster Roles
  - Take me to [Video Tutorial](https://kodekloud.com/topic/cluster-roles/)
  
In this section, we will take a look at cluster roles

## Roles
- Roles and Rolebindings are namespaced meaning they are created within namespaces.
  
  ![roles](../../images/roles.PNG)
  
## Namespaces
- Can you group or isolate nodes within  a namespace?
  - No, those are cluster wide or cluster scoped resources. They cannot be associated to any particular namespace.
  
  ![namespace](../../images/namespace.PNG)
  
- So the resources are categorized as either namespaced or cluster scoped.
  
- To see namespaced resources
  ```
  $ kubectl api-resources --namespaced=true
  ```
- To see non-namespaced resources
  ```
  $ $ kubectl api-resources --namespaced=false
  ```
  
  ![namespace1](../../images/namespace1.PNG)
  
## Cluster Roles and Cluster Role Bindings
- Cluster Roles are roles except they are for a cluster scoped resources. Kind as **`CLusterRole`** 
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    name: cluster-administrator
  rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["nodes"]
    verbs: ["get", "list", "delete", "create"]
  ```
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: cluster-admin-role-binding
  subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: cluster-administrator
    apiGroup: rbac.authorization.k8s.io
  ```
  ```
  $ kubectl create -f cluster-admin-role.yaml
  $ kubectl create -f cluster-admin-role-binding.yaml
  ```
  
 ![cr1](../../images/cr1.PNG)
  
- You can create a cluster role for namespace resources as well. When you do that user will have access to these resources across all namespaces.

#### K8s Reference Docs
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities
  
  
---

# Cluster Role

When we talked about roles and role bindings, we said that roles and role bindings are namespaced,

meaning they are created within namespaces.

If you don't specify a namespace, they are created in the default namespace and control access within

that namespace alone.

In one of the previous lectures, we discussed about namespaces and how it helps in grouping or isolating

resources like pods, deployments, and services.

But what about other resources like nodes?

Can you group or isolate nodes within a namespace? No, those are cluster wide or cluster scoped resources.

They cannot be associated to any particular namespace.

So the resources are categorized as either namespaced or cluster scoped.

we have seen a lot of Namespaced resources throughout this course, like pods and replica sets

and jobs, deployments, services, secrets.

And in the last lecture, we saw two new roles and role bindings.

These resources are created in the namespace you specify when you create them.

If you don't specify a namespace, they're created in the default namespace.

To view them or delete them or update them, you always specify the right namespace.

The cluster scoped resources are those where you don't specify a namespace when you create them like

nodes, persistent volumes persist the cluster roles and cluster role bindings 

Certificate signing request.

We saw earlier and namespace objects themselves are of course not namespaced.

To see a full list of namespaces and non namespace resources.

Run the kube control API resources command with the namespace option set.


```
kubectl api-resources --namespaced=true

kubectl api-resources --namespaced=false
```


In the previous lecture, we saw how to authorize a user to namespace resources.

We use roles and role bindings for that.

But how do we authorize users to cluster wide resources like nodes or persistent volumes?

That is where you use cluster roles and cluster role bindings.

Cluster roles are just like roles, except they are for cluster scoped resources.

Par exemple, un role cluster admin peut:
- Can view nodes
- Can create nodes
- can delete nodes

Un role storage administrator role peut:
- Can view PVs
- Can create PVs
- Can delete PVs

Create a cluster role definition file with the cluster role and specify the roles as we did before.

In this case, the resources are nodes.

Then create the cluster role.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
```

Il faut ensuite creer le cluster role
```
kubectl create -f cluster-admin-role.yaml
```

The next step is to link the user to that cluster role.

For this we create another object called cluster role.

Binding.

The role binding object links the user to the role.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```

On créer ensuite le role binding
```
kubectl create -f cluster-admin-role-binding.yaml
```

we said that cluster roles and bindings are used for cluster

scope resources, but that is not a hard rule.

You can create a cluster role for namespace resources as well.

When you do that, the user will have access to these resources across all namespaces.

Earlier, when we created a role to authorize a user to access pods, the user had access to the pods

in a particular namespace alone with cluster roles.

When you authorize a user to access the pods, the user gets access to all pods across the cluster.

Kubernetes creates a number of cluster roles by default when the cluster is first set up.

Lister les cluster role et les cluster role binding:
```
kubectl get clusterroles

kubectl get clusterrolebindings
```
