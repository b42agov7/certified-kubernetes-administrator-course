# Certificate API
  - Take me to [Video Tutorial](https://kodekloud.com/topic/certificates-api/)
  
In this section, we will take a look at how to manage certificates and certificate API's in kubernetes

## CA (Certificate Authority)
- The CA is really just the pair of key and certificate files that we have generated, whoever gains access to these pair of files can sign any certificate for the kubernetes environment.

They can create as many users as they want with whatever privileges they want.
So these files need to be protected and stored in a safe environment.

So these files need to be protected and stored in a safe environment. Say we place them on a server that is fully secure now that server becomes your CA server.

The certificate key file is safely stored in that server and only on that server. Every time you want to sign a certificate, you can only do it by logging into that server.
As of now, we have the certificates placed on the Kubernetes master node itself. So the master node is also a CA server.

The Kubeadm tool does the same thing. It creates a CA pair of files and stores that on the master node itself.

So far we have been signing requests manually, but as in when the users increase and your team grows, you need a better automated way to manage the certificates signing requests, as well as to rotate certificates when they expire.


#### Kubernetes has a built-in certificates API that can do this for you. 
- With the certificate API, we now send a certificate signing request (CSR) directly to kubernetes through an API call.
   
  ![csr](../../images/csr.PNG)

With the certificates API, you now send a certificate signing request directly to Kubernetes through an API call.
This time when the administrator receives a certificate signing request, instead of logging onto the master node and signing the certificate by himself, he creates a Kubernetes API object called CertificateSigningRequest.
Once the object is created, all CertificateSigningRequest can be seen by administrators of the cluster.
The request can be reviewed and approved easily using Kube control commands.
   
#### This certificate can then be extracted and shared with the user.
- A user first creates a key
  ```
  $ openssl genrsa -out jane.key 2048
  ```
- Generates a CSR using the key with her name on it
  ```
  $ openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr 
  ```
- Sends the request to the administrator and the adminsitrator takes the key and creates a CSR object, with kind as "CertificateSigningRequest" and a encoded "jane.csr"

The administrator takes a key and creates a CertificateSigningRequest object. The CertificateSigningRequest object
is created like any other Kubernetes object using a manifest file with the usual fields.
The kind is CertificateSigningRequest under the spec section.
In the spec section, usages defines how the certificate will be used.
Expiration seconds controls certificate validity, signer name tells Kubernetes who is allowed to sign the certificate signing request
and what kind of certificate is being requested.
The request field is where you specify the certificate signing request sent by the user, but you don't specify it as plain text.
Instead, it must be encoded using the base 64 command.
Then move the encoded text into the request field and then submit the request.

```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: jane-csr
spec:
  signerName: Kubernetes.io/kube-apisercer-client
  expirationSeconds: 600 #seconds
  usages:
  - clent auth
  request: <BASE64_ENCODED_CSR>
```

  ```
  apiVersion: certificates.k8s.io/v1beta1
  kind: CertificateSigningRequest
  metadata:
    name: jane
  spec:
    groups:
    - system:authenticated
    usages:
    - client auth
    request:
      <certificate-goes-here>
  ```

  ```
  $ cat jane.csr |base64 
  $ kubectl create -f jane.yaml
  ```
 ![csr1](../../images/csr1.PNG)

Once the object is created, all CertificateSigningRequests can be seen by administrators by running the Kube control, get csr command.

Identify the new request and approve the request by running the Kube control certificate approved command, Kubernetes signs the certificate using the CA key pairs and generates a certificate for the user.
This certificate can then be extracted and shared with the user.
  
- To list the csr's
  ```
  $ kubectl get csr
  ```

Identify the new request and approve the request by running the Kube control certificate approved command, Kubernetes signs the certificate using the CA key pairs and generates a certificate for the user. This certificate can then be extracted and shared with the user.
  
- Approve the request
  ```
  $ kubectl certificate approve jane
  ```

View the certificate by viewing it in a YAML format. The generated certificate is part of the output, but as before, it is in a base 64 and coded format.
To decode it, take the text and use the base 64 utilities decode option.
This gives the certificate in a plain text format. This can then be shared with the end user.
  
- To view the certificate
  ```
  $ kubectl get csr jane -o yaml
  ```
- To decode it
  ```
  $ echo "<certificate>" |base64 --decode
  ```
  
  ![csr2](../../images/csr2.PNG)

Now that we have seen how it works, let's see who does all of this for us.
If you look at the Kubernetes control plane, you see the kube-apiserver, the scheduler, the controller manager, etcd server, et cetera.
Which of these components is actually responsible for all the certificate related operations?


  
#### All the certificate releated operations are carried out by the controller manager. 

All the certificate related operations are carried out by the controller manager.
If you look closely at the controller manager, you will see that it has controllers in it called as CSR-Approving, CSR-Signing, et cetera, that are responsible for carrying out these specific tasks.

We know that if anyone has to sign certificates, they need the CA server's route certificate and private key. The Controller Manager service configuration has two options where you can specify these.

- If anyone has to sign the certificates they need the CA Servers, root certificate and private key. The controller manager configuration has two options where you can specify this.

  ![csr3](../../images/csr3.PNG)
  
  ![csr4](../../images/csr4.PNG)
  
  
#### K8s Reference Docs
- https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/
- https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/

### Corection
```
cat akshay.csr | base64 -w 0

controlplane ~ ➜  cat akshay-csr-k8s.yaml 
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  signerName: kubernetes.io/kube-apiserver-client
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQXlSRFBGc2ZTZUFaWUxDaUtDWlUrSElMNjRmM3RrbXVLdUpFWGNuOG9xTzRpCkhEU3hGNVIvRFVhUkNZUXpZcXVZMzhHdUFET2NDckNGN1lDK21iSVFWaUI4RDczK21LQytPa2Irc3ZhVVNLa0sKc2VWUHlWQWQ5dURCZUpyVUJVQzcrS1M1bmJPVFdoSXBkKzlpRTZQNTMxSWxqWFhiUDBwV0xWZURLZEE2bi9yNQpjUmZZdkZ4c2VHdGJUUGVESC9lU2hzb2VlOEVvU2lHUWQydVVHK0V2Uk1YZGV0UUVFWHRIQUU2cGdHNmtmUmxzCjFDSDJVcGZjNmh1aTFXL3VNM2tINjdDclJLaDVhZXgrc0psUTV2ekM1RlVhRDQ0bGhUaDhmVGlBWm5WWFkvVGIKUy9oS2p4NTJKMXl2bGJheFluRGdSSkFINHdodExrNHI5blZoamlvMDB3SURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBRURmY3Z5VjNGSHEybFBYckxOb3I1bndWVFcraGdlcXdIZUNXU2dMdjdFMEM5d0ZyaWk4CkY5Z1VRZ1RlbzRWOEpqVXZGY1dYYjgyL3RSRmVYUHhwOXhrd3ZYUXNOZ3pNSlpscExLVlVTc1g5WE81S1JOMXMKSGRYYUl4cWlIemZlYmdETFVjOTFvdDNKNU9QSTN0RElRZUd3dEp6Qnlpa1lxSno5R2FCZ3dXeEdYVlY4SXVRego0d0k5eTR5SXhBTW5BUmxySmJham9hYW14L1pZeUZxZTZHUXI0YzRtaU9ESmNsaGVHSklpelFWb0Q1MjUxWFNuCjNYcTFBY3d5TStRT3YxWFV0eTY4bmhRYXhhaStWVUpQZTlUOGdPcTd4UFRZUGJDOHUwYkVqYjJpc1BBaE4yZlkKZ3p3SExubmU0NXNvMmZCTEhIZlNxOXlRTVdoaCtuTkdSN0U9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=

controlplane ~ ➜  
```

 
  
```
kubectl certificate approve akshay

kubectl describe certificatesigningrequests agent-smith

kubectl get csr agent-smith -o yaml

kubectl delete csr agent-smith

kubectl certificate deny agent-smithj

kubectl get csr


```

