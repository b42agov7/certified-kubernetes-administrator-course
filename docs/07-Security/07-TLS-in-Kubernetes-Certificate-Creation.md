# TLS in kubernetes - Certificate Creation
  - Take me to [Video Tutorial](https://kodekloud.com/topic/tls-in-kubernetes-certificate-creation/)
  
In this section, we will take a look at TLS certificate creation in kubernetes

## Generate Certificates
- There are different tools available such as easyrsa, openssl or cfssl etc. or many others for generating certificates.

We will use openssl to generate certificates.

## Certificate Authority (CA)

- Generate Keys
  ```
  $ openssl genrsa -out ca.key 2048
  ```
- Generate CSR

The certificate signing request is like a certificate with all of your details, but with no signature.
In the certificate signing request, we specify the name of the component this certificate is for in the common name or CN field.
In this case, since we are creating a certificate for the Kubernetes CA, we name it "KUBERNETES-CA."
  ```
  $ openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
  ```

- Sign certificates

Finally, we sign the certificate using the openssl x509 command and by specifying the certificate signing request we generated in the previous command.
Since this is for the CA itself, it is self signed by the CA using its own private key that it generated in the first step.

  ```
  $ openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
  ```
 
 ![ca1](../../images/ca1.PNG)

Going forward, for all other certificates, we will use this CA key pair to sign them. The CA now has its private key and root certificate file.
 
## Generating Client Certificates

#### Admin User Certificates

- Generate Keys
We create a private key for the admin user using the OpenSSL
  ```
  $ openssl genrsa -out admin.key 2048
  ```
- Generate CSR

We then generate a CSR. And that is where we specify the name of the admin user, which is kube-admin.
A quick note about the name. It doesn't really have to be kube-admin, it could be anything.
But remember, this is the name that kube control client authenticates with and when you run the kubectl command.
So in the audit logs and elsewhere, this is the name that you will see.

So provide a relevant name in this field.

  ```
  $ openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
  ```

- Sign certificates

Finally, generate a signed certificate using the openssl x509 command.
But this time, you specify the CA certificate and the CA key.

You're signing your certificate with the CA pair.
That makes this a valid certificate within your cluster.
The signed certificate is then output to admin.crt file.
That is the certificate that the admin user will use to authenticate to Kubernetes cluster.

  ```
  $ openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
  ```

If you look at it, this whole process of generating a key and a certificate pair is similar to creating a user account for a new user.
The certificate is the validated user ID and the key is like the password.
  
  ![ca2](../../images/ca2.PNG)
  
  
- Certificate with admin privilages

How do you differentiate this user from any other users?
The user account needs to be identified as an admin user, and not just another basic user.
You do that by adding the group details for the user in the certificate.
In this case, a group named "SYSTEM:MASTERS" in system Kubernetes with administrative privileges.

  ```
  $ openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
  ```
  
#### We follow the same procedure to generate client certificate for all other components that access the kube-apiserver.

Now, the kube-scheduler is a system component part of the Kubernetes control pane.
So its name must be prefixed with the keyword SYSTEM.

  ![crt1](../../images/crt1.PNG)

The same with kube controller manager, it is again, a system component.
So its name must be prefixed with the keyword system.
  
  ![crt2](../../images/crt2.PNG)

And finally, kube-proxy.
  
  ![crt3](../../images/crt3.PNG)

We will follow the same procedure to create the remaining three client certificates for API servers and kubelets when we create the server certificates for them.
   
  ![crt4](../../images/crt4.PNG)

Now what do you do with these certificates?
Take the admin certificate, for instance, to manage the cluster, you can use this certificate instead of a username and password in a rest API call you make to the Kube-API server, you specify the key, the certificate and the CS certificate as options.
That's one simple way.
The other way is to move all of these parameters into a configuration file called kube-config within that specify the API server endpoint details, the certificates to use, et cetera.
That is what most of the Kubernetes clients use.
  
## Generating Server Certificates

Remember in the prerequisite lecture, we mentioned that for clients to validate the certificate sent by the server and vice versa, they all need a copy of the certificate authorities public certificate the one that we said is already installed
within the users browsers in case of a web application.

Similarly, in kubernetes for these various components to verify each other, they all need a copy of the CA's root certificate.
So whenever you configure a server or a client with certificates, you will need to specify the ca root certificate as well.

## ETCD Server certificate

We follow the same procedure as before to generate a certificate for etcd. We will name it ETCD-SERVER.

ETCD-SERVER can be deployed as a cluster across multiple servers as in a high availability environment.
In that case, to secure communication between the different members in the cluster, we must generate additional peer certificates. Once the certificates are generated, specify them while starting the etcd server.

  ![etc1](../../images/etc1.PNG)
  
  ![etc2](../../images/etc2.PNG)

There are key and cert file options where you specify the etcd server keys. There are other options available for specifying the peer certificates.
And finally, as we discussed earlier, it requires the CA root certificate to verify that the clients connecting to the etcd server are valid.
  
## Kube-apiserver certificate

  ![api1](../../images/api1.PNG)

Everyone talks to the Kube-API server, every operation goes through the Kube-API server.
Anything moves within the cluster, the API server knows about it.
You need information, you talk to the API server.
And so it goes by many names and aliases within the cluster.
Its real name is Kube-API server.
But some call it Kubernetes, because for a lot of people who don't really know what goes under the hoods of Kubernetes, the Kube-API server is Kubernetes.
Others like to call it Kubernetes.default.
Well, some refer to it as Kubernetes.default.svc, and some like to call it by its full name, Kubernetes.default.svc.cluster.local.
Finally, it is also referred to in some places simply by its IP address, the IP address of the host running the Kube-API server, or the pod running it. So all of these names must be present in the certificate generated for the Kube-API server.
Only then those referring to the Kube-API server by these names will be able to establish a valid connection.
So we use the same set of commands as earlier to generate a key in the certificate signing request, you specify the name Kube-API server.
But how do you specify all the alternate names?
For that, you must create an OpenSSL config file.
Create an openssl.cnf file and specify the alternate names in the alt names section of the file.
Include all the DNS names the API server goes by as well as the IP address.
Pass this config file as an option while generating the certificate signing request.
Finally, sign the certificate using the CA certificate and key you then have the Kube-API server certificate.
  
  ![api2](../../images/api2.PNG)

It is time to look at where we are going to specify these keys. Remember to consider the API client certificates
that are used by the API server while communicating as a client to the etcd and kubelet servers.
The location of these certificates are passed in to the Kube-API servers executable or service configuration file.
First, the CA file needs to be passed in.
Remember, every component needs the CA certificate to verify its clients. Then we provide the API server certificates under the TLS cert options. We then specify the client certificates used by Kube-API server to connect to the etcd server, again, with the CA file.
And finally, the Kube-API server client certificates to connect to the kubelets.

## Kubelet server certificate

The kubelet server is an HTTPS API server that runs on each node responsible for managing the node.

That's who the API server talks to to monitor the note as well as send information regarding what parts to schedule on this node.

As such, you need a key certificate pair for each node in the cluster.
Now, what do you name these certificates?
Are they all going to be named kubelets?
No, they will be named after their nodes, node01, node02, and node03.

Once the certificates are created, use them in the kubelet config file. As always, you specify the root CA certificate, and then provide the kubelet node certificates.
You must do this for each node in the cluster.
We also talked about a set of client certificates that will be used by the kubelet to communicate with a Kube-API server.
These are used by the kubelet to authenticate into the Kube-API server. They need to be generated as well. What do you name these certificates? The API server needs to know which node is authenticating and give it the right set of permissions.
So it requires the nodes to have the right names in the right formats. Since the nodes are system components like the kube-scheduler and the controller manager we talked about earlier, the format starts with the system keyword, followed by node and then the node name, in this case node01 to node03.
And how would the API server give it the right set of permissions?
Remember, we specified a group name for the admin user so the admin user gets administrative privileges?

Similarly, the nodes must be added to a group named system nodes.
Once the certificates are generated, they go into the kube-config files as we discussed earlier.


## Kubectl Nodes (Server Cert)

   ![kctl1](../../images/kctl1.PNG)
   
## Kubectl Nodes (Client Cert)

   ![kctl2](../../images/kctl2.PNG)
   
   
   
  
  

  

  


  
  
  
  
 
