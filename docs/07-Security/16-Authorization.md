# Authorization
  - Take me to [Video Tutorial](https://kodekloud.com/topic/authorization/)
  
In this section, we will take a look at authorization in kubernetes

## Why do you need Authorization in your cluster?
- As an admin, you can do all operations
  ```
  $ kubectl get nodes
  $ kubectl get pods
  $ kubectl delete node worker-2
  ```
  
  ![at1](../../images/at1.PNG)
  
## Authorization Mechanisms
- There are different authorization mechanisms supported by kubernetes
  - Node Authorization
  - Attribute-based Authorization (ABAC)
  - Role-Based Authorization (RBAC)
  - Webhook
  
## Node Authorization

  ![node-auth](../../images/node-auth.png)
  
## ABAC

  ![abac](../../images/abac.PNG)
  
## RBAC

  ![rbac](../../images/rbac.PNG)

## Webhook
  
  ![webhook](../../images/webhook.PNG)
  
## Authorization Modes
- The mode options can be defined on the kube-apiserver

  ![mode](../../images/mode.PNG)
  
- When you specify multiple modes, it will authorize in the order in which it is specified

  ![mode1](../../images/mode1.PNG)
  
  
  #### K8s Reference Docs
  - https://kubernetes.io/docs/reference/access-authn-authz/authorization/
  


---


Les autorisations permettent de définir ce que l'on peut faire dans un cluster une fois qu'on y accès via les authorisations.

Pourquoi avons nous besoin des autorisations dans kubernetes ?

Un administrateur doit pouvoir réaliser plusisuers types d'actions dans un cluster kubernetes.

```
kubectl get pods

kubectl get nodes

kubectl delete node worker-2
```

Dans un cluster, il n'y a pas que l'admin qui y accès, on peut avoir d'autres utilisateurs comme des dev, des testeurs ou des applications de monitoring, bots.

Pour ces utilisateurs, on va devoir créer des comptes, mais ces comptes ne doivent pas avoir les mêmes accès que nous, comme ajouter ou supprimer des nodes ou encore modifier des config réseaux ou de storage.

Il en vas de même pour les services accounts (bots).

Lorsqu'on partage notre cluster avec d'autres organisations ou teams en le partionant de manière logique (namespaces), on veut aussi restreindre l'accès à ces namespaces au personnes concernés.

Les authorisations peuvent nous aider à résoudre ce problème.

Dans Kubernetes, il existe différents type de méchanisme d'autorisations.
- Node authorizer
- ABAC
- RBAC
- Webhook

## Node Authorizer

Un Kubelet intéragit avec le kube-apiserver et donc avec le master node.

Ces actions sont:

Read
- Services
- Endpoints
- Nodes
- Pods

Write
- Node status
- Pod Status
- events

Comme nous le savons, un node du cluster possède un certificat pour intéragit avec ce dernier et dans le nom du certificat il y "system:node:node01", donc les nodes avec ce nom sont authorisé par le Node Authorizer à pouvoir réaliser les actions cités précédement.


## ABAC

For instance, a user attribute based authorization is where you associate a user or a group of users
with a set of permissions.
In this case, we say the dev user can view, create, and delete pods.
You do this by creating a policy file with a set of policies defined in a JSON format.

```
{"kind": "Policy", "spec": {"user": "dev-user", "namespace": "*", "resource": "pods", "apiGroup": "*"}}
{"kind": "Policy", "spec": {"user": "dev-user-2", "namespace": "*", "resource": "pods", "apiGroup": "*"}}
{"kind": "Policy", "spec": {"group": "dev-users", "namespace": "*", "resource": "pods", "apiGroup": "*"}}
{"kind": "Policy", "spec": {"user": "security-1", "namespace": "*", "resource": "csr", "apiGroup": "*"}}
```

This way you pass this file into the API server.
Similarly, we create a policy definition file for each user or group in this file.
Now, every time you need to add or make a change in the security, you must edit this policy file manually
and restart the kube API server.
As such, the attribute based access control configurations are difficult to manage.
We will look at role based access controls.


## RBAC

Role based access controls make this much easier with role based access controls.
Instead of directly associating a user or a group with a set of permissions.
We define a rule in this case for developers.
We create a rule with the set of permissions required for developers.
Then we associate all the developers to that role.
Similarly, create a role for security users with the right set of permissions required for them.
Then associate the user to that role going forward whenever a change needs to be made to the user's
access.
We simply modify the role and it reflects on all developers immediately.
Role based access controls provide a more standard approach to managing access within the Kubernetes
cluster.

## Webhook

Now what if you want to outsource all the authorization mechanisms?

Say you want to manage a authorization externally and not through the built in mechanisms that we just

discussed.

For instance, Open Policy Agent is a third party tool that helps with admission control and authorization.

You can have Kubernetes make an API call to the open policy agent with the information about the user

and his access requirements, and have the open policy agent decide if the user should be permitted

or not.

Based on that response, the user is granted access.

## Authorization Mode

Il existe 2 modes.

- AlwaysAllow
- AlwaysDeny

Always allow allows all requests without performing any authorization checks.

Always deny denies all requests.

So where do you configure these modes?

Which of them are active by default?

Can you have more than one at a time?

How does authorization work?

If you do have multiple ones configured, the modes are set using the authorization mode option on the

kube API server.

If you don't specify this option, it is set to always allow by default.


```
$ ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address=${INTERNAL_IP} \
  --allow-privileged=true \
  --apiserver-count=3 \
  --authorization-mode=AlwaysAllow \
  --bind-address=0.0.0.0 \
  --enable-swagger-ui=true \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt \
  --etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key \
  --etcd-servers=https://127.0.0.1:2379 \
  --event-ttl=1h \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/apiserver-etcd-client.crt \
  --kubelet-client-key=/var/lib/kubernetes/apiserver-etcd-client.key \
  --service-node-port-range=30000-32767 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --tls-cert-file=/var/lib/kubernetes/apiserver.crt \
  --tls-private-key-file=/var/lib/kubernetes/apiserver.key \
  --v=2
```

You may provide a comma separated list of multiple modes that you wish to use.

In this case, I want to set it to node, Rbac and webhook.

When you have multiple modes configured, your request is authorized using each one in the order it

is specified.

```
$ ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address=${INTERNAL_IP} \
  --allow-privileged=true \
  --apiserver-count=3 \
  --authorization-mode=Node,RBAC,Webhook \
  --bind-address=0.0.0.0 \
  --enable-swagger-ui=true \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/apiserver-etcd-client.crt \
  --etcd-keyfile=/var/lib/kubernetes/apiserver-etcd-client.key \
  --etcd-servers=https://127.0.0.1:2379 \
  --event-ttl=1h \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/apiserver-etcd-client.crt \
  --kubelet-client-key=/var/lib/kubernetes/apiserver-etcd-client.key \
  --service-node-port-range=30000-32767 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --tls-cert-file=/var/lib/kubernetes/apiserver.crt \
  --tls-private-key-file=/var/lib/kubernetes/apiserver.key \
  --v=2
```

For example, when a user sends a request, it's first handled by the node authorizer.

The node authorizer handles only node requests, so it denies the request.

Whenever a module denies a request, it is forwarded to the next one in the chain.

The role based access control module performs its checks and grants.

The user permission.

Authorization is complete and user is given access to the requested object.

So every time a module denies a request, it goes to the next one in the chain, and as soon as a module

approves the request, no more checks are done and the user is granted permission.
