# TLS in Kubernetes
  - Take me to [Video Tutorial](https://kodekloud.com/topic/tls-in-kubernetes/)
  
In this section, we will take a look at TLS in kubernetes

#### The two primary requirements are to have all the various services within the cluster to use server certificates and all clients to use client certificates to verify they are who they say they are.
- Server Certificates for Servers
- Client Certificates for Clients

  ![tls](../../images/tls.PNG)
  
#### Let's look at the different components within the k8s cluster and identify the various servers and clients and who talks to whom.

  ![certs](../../images/certs.PNG)
  
In the world of certificates we have: 

- Server Certificates, used by servers to secure connections using private keys and certificates (public key) (configured on the servers)
- Root Certificates, which define The Certificate Authority (CA), which has its own private and public key, the latter allows you to sign the certificates of the servers (configured on the CA server)
- Client Certificates that allow a server to verify the client's identity (configured on the clients)

## How these concepts are present in a Kubernetes cluster

A Kubernetes cluster is a set of moster nodes and worker nodes. All communications between these nodes must be secure and encrypted. Similarly, all interactions with clients and services must be secure and encrypted. For example, an admin who interacts with the kube-apiserver via kubectl must establish a secure TLS connection. All communication between Kubernetes components must be secure.

To do this, every component/server must have Server Certificates and all clients must have Client Certificates in order to prove what they claim to be.

## Server Certificates for Servers

As we know already, the API server exposes an HTTPS service that other components, as well as external users, use to manage the Kubernetes cluster.

So it is a server and it requires certificates to secure all communication with its clients.

So we generate a certificate and key pair. We call it apiserver.crt and apiserver.key.


Another server in the cluster is the etcd-server. The etcd-server stores all information about the cluster, so it requires a pair of certificate and key for itself. We will call it etcd-server.crt and etcd-server.key.


The other server component in the cluster is on the worker nodes.
They are the kubelet-services. They also expose an HTTPS API endpoint that the kube-apiserver talks to, to interact with the worker nodes. Again, that requires a certificate and key pair. We call it kubelet.crt and kubelet.key.

## Client Certificates for Clients

Who are the clients who access these services? 

The clients who access the kube-apiserver are us, the administrators, through kubectl or REST API.

The admin user requires a certificate and key pair to authenticate to the kube-apiserver. We will call it admin.crt and admin.key.

The scheduler talks to the kube-apiserver to look for pods that require scheduling and then get the API server to schedule the pods on the right worker notes.
The scheduler is a client that accesses the kube-apiserver.


As far as the kube-apiserver is concerned, the scheduler is just another client like the admin user. so the scheduler needs to validate its identity using a client TLS certificate,

so it needs its own pair of certificate and keys.
We will call it scheduler.crt and scheduler.key.

The kube-controller-manager is another client that accesses the kube-apiserver, so it also requires a certificate for authentication to the kube-apiserver, so we create a certificate pair for it.

The last client component is the kube-proxy. The kube-proxy requires a client certificate to authenticate to the kube-apiserver, and so it requires its own pair of certificate and keys.

We will call them kube-proxy.crt and kube-proxy.key.

The servers communicate amongst them as well.

For example, the kube-apiserver communicates with the etcd-server. In fact, of all the components, the kube-apiserver is the only server that talks to the etcd-server.

So as far as the etcd-server is concerned, the kube-apiserver is a client, so it needs to authenticate.

The kube-apiserver can use the same keys that it used earlier for serving its own API service, the apiserver.crt and the apiserver.key files. Or you can generate a new pair of certificates specifically for the kube-apiserver to authenticate to the etcd-server.

The kube-apiserver also talks to the kubelet server on each of the individual nodes. That's how it monitors the worker nodes. For this, again, it can use the original certificates or generate new ones specifically for this purpose.



So that's too many certificates. Let's try and group them.
There are a set of client certificates mostly used by clients to connect to the kube-apiserver, and there are a set of server side certificates used by the kube-apiserver, etcd-server, and kubelet to authenticate their clients.

We will now see how to generate these certificates.

As we know already, we need a certificate authority to sign all of these certificates.
Kubernetes requires you to have at least one certificate authority for your cluster.
In fact, you can have more than one, one for all the components in the cluster and another one specifically for etcd.


In that case, the etcd-server's certificates and the etcd-server's client certificates, which in this case is the apiserver client certificate, will be all signed by the etcd-server CA.

For now, we will stick to just one CA for our cluster. The CA, as we know, has its own pair of certificates and key. We will call it ca.crt and ca.key.
