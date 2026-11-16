# Kubernetes Security Primitives
  - Take me to [Video Tutorial](https://kodekloud.com/topic/kubernetes-security-primitives/)
  
In this section, we will take a look at kubernetes security primitives

## Secure Hosts

All access to these hosts must be secured root access disabled, password based authentication disabled, and only SSH key based authentication to be made available.

 ![sech](../../images/sech.PNG)

 kube-apiserver is at the center of operations in Kubernetes, so it's something we need to secure. We interact with it via kubectl or directly via its API.
 So that's the first line of defense.
  
## Secure Kubernetes
- We need to make two types of decisions.
  - Who can access?
  - What can they do?
 
  ![seck](../../images/seck.PNG)
  
## Authentication
- Who can access the API Server is defined by the Authentication mechanisms.

There are different kind of authentications mechanisms.
- Static Token File
- Certificates
- External Authentication providers - LDAP
- Service Accounts
  
## Authorization
- Once they gain access to the cluster, what they can do is defined by authorization mechanisms.

Authorization is implemented by:
- RBAC Authorization
- ABAC Authorization
- Node Authorization
- Webhook Mode

## TLS Certificates
- All communication with the cluster, between the various components such as the ETCD Cluster, kube-controller-manager, scheduler, api server, as well as those running on the working nodes such as the kubelet and kubeproxy is secured using TLS encryption.

 ![tls](../../images/tls.PNG)
 
## Network Policies
What about communication between applications within the cluster?

By default, all pods can access other pods in the cluster.

Access between them can be restricted through network policies

  ![np](../../images/np.PNG)
  
