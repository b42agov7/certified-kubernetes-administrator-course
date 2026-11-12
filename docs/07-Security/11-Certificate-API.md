# Certificate API
  - Take me to [Video Tutorial](https://kodekloud.com/topic/certificates-api/)
  
In this section, we will take a look at how to manage certificates and certificate API's in kubernetes

## CA (Certificate Authority)
- The CA is really just the pair of key and certificate files that we have generated, whoever gains access to these pair of files can sign any certificate for the kubernetes environment.

#### Kubernetes has a built-in certificates API that can do this for you. 
- With the certificate API, we now send a certificate signing request (CSR) directly to kubernetes through an API call.
   
  ![csr](../../images/csr.PNG)
   
#### This certificate can then be extracted and shared with the user.
- A user first creates a key
  ```
  $ openssl genrsa -out jane.key 2048
  ```
- Generates a CSR
  ```
  $ openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr 
  ```
- Sends the request to the administrator and the adminsitrator takes the key and creates a CSR object, with kind as "CertificateSigningRequest" and a encoded "jane.csr"
  ```
  apiVersion: certificates.k8s.io/v1beta1
  kind: CertificateSigningRequest
  metadata:
    name: jane
  spec:
    groups:
    - system:authenticated
    usages:
    - digital signature
    - key encipherment
    - server auth
    request:
      <certificate-goes-here>
  ```
  $ cat jane.csr |base64 
  $ kubectl create -f jane.yaml
  ```
 ![csr1](../../images/csr1.PNG)
  
- To list the csr's
  ```
  $ kubectl get csr
  ```
- Approve the request
  ```
  $ kubectl certificate approve jane
  ```
- To view the certificate
  ```
  $ kubectl get csr jane -o yaml
  ```
- To decode it
  ```
  $ echo "<certificate>" |base64 --decode
  ```
  
  ![csr2](../../images/csr2.PNG)
  
#### All the certificate releated operations are carried out by the controller manager. 
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

