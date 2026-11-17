# View Certificate Details
  - Take me to [Video Tutorial](https://kodekloud.com/topic/view-certificate-details/)
  
In this section, we will take a look how to view certificates in a kubernetes cluster.

## View Certs 

First of all, it's important to know how the cluster was set up.
There are different solutions available for deploying a Kubernetes cluster, and they use different methods to generate and manage certificates. If you were to deploy a Kubernetes cluster from scratch, you generate all the certificates by yourself as we did in the previous lecture.
Or else, if you were to rely on an automated provisioning tool like kube ADM, it takes care of automatically generating and configuring the cluster for you while you deploy all the components as native services on the nodes in the hard way. The kubectl tool deploys these as pods.
So it's important to know where to look at to view the right information.

```
cat /etc/systemd/system/kube-apiserver.service
```

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```


 ![hrd](../../images/hrd.PNG)

 ![hrd1](../../images/hrd1.PNG)
 
 - To view the details of the certificate
   ```
   $ openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
   ```
   
   ![hrd2](../../images/hrd2.PNG)
   
#### Follow the same procedure to identify information about of all the other certificates

   ![hrd3](../../images/hrd3.PNG)
   
## Inspect Server Logs - Hardware setup


- Inspect server logs using journalctl
  ```
  $ journalctl -u etcd.service -l
  ```
  
  ![hrd4](../../images/hrd4.PNG)
  
## Inspect Server Logs - kubeadm setup
- View logs using kubectl
  ```
  $ kubectl logs etcd-master
  ```
  ![hrd5](../../images/hrd5.PNG)
  
- View logs using docker ps and docker logs
  ```
  $ docker ps -a
  $ docker logs <container-id>
  ```
  ![hrd6](../../images/hrd6.PNG)
  
#### K8s Reference Docs
- https://kubernetes.io/docs/setup/best-practices/certificates/#certificate-paths
  
  

  

   
   

