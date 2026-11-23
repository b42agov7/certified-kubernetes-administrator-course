# RBAC
  - Take me to [Video Tutorial](https://kodekloud.com/topic/role-based-access-controls/)

In this section, we will take a look at RBAC

## How do we create a role?
- Each role has 3 sections
  - apiGroups
  - resources
  - verbs
- create the role with kubectl command
  ```
  $ kubectl create -f developer-role.yaml
  ```

## The next step is to link the user to that role.
- For this we create another object called **`RoleBinding`**. This role binding object links a user object to a role.
- create the role binding using kubectl command
  ```
  $ kubectl create -f devuser-developer-binding.yaml
  ```
- Also note that the roles and role bindings fall under the scope of namespace.
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: developer
  rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "list", "update", "delete", "create"]
  - apiGroups: [""]
    resources: ["ConfigMap"]
    verbs: ["create"]
  ```
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: devuser-developer-binding
  subjects:
  - kind: User
    name: dev-user # "name" is case sensitive
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: Role
    name: developer
    apiGroup: rbac.authorization.k8s.io
  ```
  ![rbac1](../../images/rbac1.PNG)
  

## View RBAC
  
- To list roles
  ```
  $ kubectl get roles
  ```
- To list rolebindings
  ```
  $ kubectl get rolebindings
  ```
- To describe role 
  ```
  $ kubectl describe role developer
  ```
  
  ![rbac2](../../images/rbac2.PNG)
    
- To describe rolebinding
  ```
  $ kubectl describe rolebinding devuser-developer-binding
  ```
  
  ![rbac3](../../images/rbac3.PNG)
  
#### What if you being a user would like to see if you have access to a particular resource in the cluster.
## Check Access

- You can use the kubectl auth command
  ```
  $ kubectl auth can-i create deployments
  $ kubectl auth can-i delete nodes
  ```
  ```
  $ kubectl auth can-i create deployments --as dev-user
  $ kubectl auth can-i create pods --as dev-user
  ```
  ```
  $ kubectl auth can-i create pods --as dev-user --namespace test
  ```
  
  ![rbac5](../../images/rbac5.PNG)
  
## Resource Names
- Note on resource names we just saw how you can provide access to users for resources like pods within the namespace.
  ```
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: developer
  rules:
  - apiGroups: [""] # "" indicates the core API group
    resources: ["pods"]
    verbs: ["get", "update", "create"]
    resourceNames: ["blue", "orange"]
  ```  
  ![rbac4](../../images/rbac4.PNG)
  
#### K8s Reference Docs
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/
- https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities

---

RBAC

Comment créer un role? Nous créeons d'abord un role definition file.

So we create a role definition file with the API version set to Rbac, dot authorization, dot k8s dot io

v1, and kind set to role.

We name the role developer as we are creating this role for developers.

Nous voulons que le developpeurs puisse:
- Can view PODs   
- Can create PODs   
- Can delete PODs   
- Can create ConfigMaps   

And then we specify rules.

Each rule has three sections API groups, resources and verbs.

The same things that we talked about in one of the previous lectures.

For core group.

You can leave the API groups section as blank.

For any other group, you specify the group name.

The resources that we want to give developers access to are pods.

The actions that they can take are list, get, create, and delete.

Similarly, to allow the developers to create config maps, we add another rule to create config map.

We can add multiple rules for a single role like this.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```

Pour créer le role, nous utilison la commande ci-dessous:
```
kubectl create -f developer-role.yaml
```

L'étape suivante consiste à link le user au role. Nous créons on nouvel object, rule binding.

The rule binding object links a user object to a rule.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```


The rule binding object links a user object to a role.

We will name it devuser

The kind is role binding.

It has two sections.

The subjects is where we specify the user details.

The roleref section is where we provide the details of the rule we created.

Pour créer le role binding, on utilise:
```
kubectl create -f devuser-developer-binding.yaml
```

Also note that the roles and role bindings fall under the scope of namespaces.

So here the dev user gets access to pods and Configmaps within the default namespace.

If you want to limit the dev users access within a different namespace, then specify the namespace

within the metadata of the definition file while creating them.


Lister les roles
```
kubectl get roles
```

Pour lister les rolebinding
```
kubectl get rolebindings
```

Pour avoir plus de détail à propos d'un role
```
kubectl describe role devloper
```

Pour avoir plus de détail à propos d'un role binding
```
kubectl describe rolebinding devuser-devloper-binding
```

What if you, being a user, would like to see if you have access to a particular resource in the cluster?
```
kubectl auth can-i create deployments

kubectl auth can-i delete nodes
```

If you are an administrator, then you can even impersonate another user to check their permission.

```
kubectl auth can-i create deployments --as dev-user
```

```
kubectl auth can-i create pods --as dev-user --namespace test
```


Il est possible de donner des accès a des ressources spécifique, il faut utiliser le champs resourceNames

Imaginons nous avons 5 pods;
- blue
- orange,
- green,
- purple,
- pink

Si nous voulons que le dev ne puisse intéragir qu'avec les pods blue et orange:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "create", "delete"]
  resourceNames: ["blue", "orange"]
```
