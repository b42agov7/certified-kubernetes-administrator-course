# Service Account
  - Take me to [Video Tutorial](https://kodekloud.com/topic/service-account/)
  
In this section, we will explore **Service Accounts** in Kubernetes and understand their significance and usage.

### Why Do You Need a Service Account in Your Cluster?

Kubernetes operates with two main types of users:

1. **Human Users**: These are users who interact with the Kubernetes cluster through tools like `kubectl`.
2. **Bots/Applications**: These include services like Prometheus, Jenkins, and other system applications.


Service accounts are primarily used to authenticate these bots or applications to the Kubernetes API server.

### Service Account Creation

![alt text](../../images/sa.png)




### Service Accounts and Authentication



When you want to access the Kubernetes API from within a pod, you need to authenticate the pod with the API server. **Service accounts** are used for this authentication process.

When a service account is created in Kubernetes, a secret token is also created. This token can be used as a bearer token that can be used to authenticate   to the API server. By default, this token is mounted as a volume in the pod and is accessible at the following path within the pod: `/var/run/secrets/kubernetes.io/serviceaccount/token`.
 

![alt text](../../images/sa2.png)


By default, every Kubernetes namespace has a `default` service account. Every pod in that namespace is automatically assigned this `default` service account unless otherwise specified. If you want to use a different service account for a specific pod, you can specify it in the pod specification.

here’s an example of how you can specify a service account for a pod in the pod specification:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  serviceAccountName: myapp-service-account
  
```

to opt out of  serviceaccount automounting

```yaml
spec:
  automountServiceAccountToken: false
```
 


### Enhanced Security Features (Kubernetes v1.22+)

These service account tokens do not have expiration or audience restrictions by default. However, Kubernetes v1.22 and later introduced improvements to enhance token security.

- Audience Bound: The token is restricted to a specific audience (e.g., the Kubernetes API server).
- Time Bound: The token has an expiration time (default of 1 hour).
- Object Bound: The token is bound to specific objects, such as a pod or service account.

thereby enhanced security by limiting the scope of the token's validity and lifecycle.

so in v1.22, the TokenRequest API was introduced to request a token with a specific audience and expiration time. This API is used for obtaining tokens instead of using service account token Secret objects. Tokens obtained from the TokenRequest API are more secure than those stored in Secret objects because they have a bounded lifetime and are not readable by other API clients.

also in this version  the service account injected as projected volume as shown below

 

![alt text](../../images/sa3.png)

 

In kubernetes. io/service-account-token type of Secret is used to store a tokencredential that identifies a service account.
Since v1.22, this type of Secret is no longer used to mount credentials into Pods, and obtaining tokens via the TokenRequest API is recommended instead of using service account token Secret objects. Tokens obtained from the TokenRequest API are more secure than ones stored in Secret objects, because they have a bounded lifetime and are not readable by other API clients. You can use the kubectl create token command to obtain a token from the TokenRequest API. 

### Creating Service Account Tokens
In Kubernetes v1.24, KEP-2799 introduced the reduction of automatically created token secrets for service accounts. You must now manually create service account token secrets using the kubectl create token command.

```bash
 kubectl create token <serviceaccount>
```

![alt text](../../images/sa4.png)


You should only create a service account token Secret object if you can't use the TokenRequest API to obtain a token, and the security exposure of persisting a non-expiring token credential in a readable API object is
acceptable to you. for that 

- Create a service account.
- Create a secret for the service account.
- In secret add an annotation `kubernetes.io/service-account.name: <serviceaccountname>` .

 

![alt text](../../images/sa5.png)

 
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
 
  
  ---

  Dans kubernetes, il existe 2 types de compatible

- User account, utilié par des humains (admin, developer)
- Service account, utilisé par des machines (peut être une application qui intéragit avec le cluster kubernetes prometheus ou jenkins)


Un service account est l'identité d'un compte, il permet au service de pouvoir s'identifier au pres du cluster.

A service account is associated with what is known as a token.

A token is what is used to authenticate a service account into the Kubernetes API.

So if you have ever authenticated to an API using a curl command or something that you probably know

of, the tokens that are passed through as a bearer token in the header of the call.

This token is what proves the identity of the service account to the API endpoint.


By default, when a Kubernetes cluster is set up, it creates a service account by the name default

in all namespaces.

pour lister les services accounts:
```
kubectl get serviceaccount
```
To see the details of an existing service account:
```
kubectl describe serviceaccount default
```

Whenever a pod is created, the default service account is automatically attached to the pod.
```
kubectl describe pod my-pod
```

So the service account gets mounted as a projected volume within the pod.

So think of a projected volume as a dynamic directory created inside the pod by Kubernetes automatically.

So it is mounted at the location:
`/var/run/secrets/kubernetes.io/serviceaccount`

```
                                 +-------------------------------+
                                 |              Pod              |
                                 |                               |
                                 |    my-kubernetes-dashboard    |
                                 |                               |
                                 |  +-------------------------+  |
                                 |  |    /var/run/secrets     |  |
                                 |  |     /kubernetes.io      |  |
                                 |  |     /serviceaccount     |  |
                                 |  +-------------------------+  |
                                 |               |               |
+-------------------------+      |               |               |
| Service Account         |      |            .-----.            |
|                         |      |           (_______)           |
|         Default --------+------+---------- |_______|           |
|                         |      |           |_______|           |
|                         |      |           (_______)           |
+-------------------------+      +-------------------------------+
```



Now if you list the contents of this directory, you'll see the token available as a file, and if you

look inside the contents of the file, you'll see the actual token itself.

```
kubectl exec -it my-kubernetes-dashboard ls /var/run/secrets/kubernetes.io/serviceaccount
```

The default service account comes with a lot of limitations, so if you have specific access requirements

we must create a custom service account.

## Create a Service Account

```
kubectl create serviceaccount my-account-name

kubectl create serviceaccount dashboard-sa
```

or in a declarative way:
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: default
```

to view service accounts:
```
kubectl get serviceaccount
```

to list details of a service account:
```
kubectl describe serviceaccount dashboard-sa
```

As you can see, when you first create a service account, it creates a blank service account with most

of its properties set to none.

So to associate a newly created service account to a pod, we pass the name of the service account using

the service account name field in the pod spec.

```
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa
```

This time when we describe the pod, we see the dashboard service account associated with it.

And if you look inside the mounted directory, you get to see the token file.

Kubernetes automatically creates a short lived token and mounts it inside a projected volume.

The Kubelet also automatically rotates this token, and this token is bound to the life of the pod,

so it is automatically expired when the pod is deleted.

Now, if you don't want a token to be automatically created and mounted inside the pod for the service

account, you may add the auto mount Service account token key to false like this.

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: default
automountServiceAccountToken: false
```

You can define this either at the service account level.

That way, no pods that use the service account gets the service account token auto mounted.


Or you can define it at the pod definition level so that irrespective of whether this property is set

at the service account level, it won't be auto mounted for that pod.
```
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa
  automountServiceAccountToken: false
```
Now, for the pods or applications hosted within the Kubernetes cluster.

This setup that we've discussed so far works fine.

So the pod is associated to a service account, and as soon as the pod is associated to a service account,

Kubernetes automatically creates a token with an expiry date and automatically mounts that within the

pod at a particular location, which the application running inside the pod can refer to and read from

to make API calls to Kubernetes.

At times, you might want to create a token that is to be used outside of the cluster, maybe to paste

into the Kubernetes dashboard that we had created, or to put it into a CI CD tool or an external monitoring

tool.

In such cases, you can create a token by running the kubectl create token command.

```
kubectl create token dashboard-sa
```

It creates a token and shares it on screen.

And this token is not stored in any secret.

By default, all tokens are valid for one hour to extend the validity.

Add a duration flag to the command.

```
kubectl create token dashboard-sa --duration 2h
```

If you decode this token using the jq and the base64 utility, you'll see an expiry defined as well

as the service account name and other details that are associated with this token.

This token can then be used as an authentication bearer token while making a Rest call to the Kubernetes

API.

```
jq -R 'split(".") | select(length > 0) | .[0],.[1] | @base64d | fromjson' <<< eyJhbGciOiJSUz...
```

Summary   
Service Accounts are used by other applications or services to interact with Kubernetes   

Tokens are created for service accounts to prove identity.   

To create a new service account run   

```
kubectl create serviceaccount dashboard-sa
```

To use the Service Account from an external application (CI/CD, monitoring tools)

```
kubectl create token dashboard-sa --duration 2h
```

Every namespace has a default service account   

The default service account is automatically attached to the pod on creation.   

To attach a service account to a pod use serviceAccountName field   

When a service account is attached to a pod, Kubernetes:   

Automatically creates a token and mounts as a projected volume   
Automatically rotates token   
Automatically expires then token when the pod is deleted  
