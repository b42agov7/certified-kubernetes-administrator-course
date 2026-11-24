# Image Security
  - Take me to [Video Tutorial](https://kodekloud.com/topic/image-security/)

In this section we will take a look at image security

# Image
   
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
    - name: nginx
      image: nginx
  ```
  
  ![img1](../../images/img1.PNG)
  
  ![img2](../../images/img2.PNG)
  
# Private Registry
- To login to the registry
  ```
  $ docker login private-registry.io
  ```
- Run the application using the image available at the private registry
  ```
  $ docker run private-registry.io/apps/internal-app
  ```
  
  ![prvr](../../images/prvr.PNG)
  
- To pass the credentials to the docker untaged on the worker node for that we first create a secret object with credentials in it.
  ```
  $ kubectl create secret docker-registry regcred \
    --docker-server=private-registry.io \ 
    --docker-username=registry-user \
    --docker-password=registry-password \
    --docker-email=registry-user@org.com
  ```
- We then specify the secret inside our pod definition file under the imagePullSecret section 
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx-pod
  spec:
    containers:
    - name: nginx
      image: private-registry.io/apps/internal-app
    imagePullSecrets:
    - name: regcred
  ```
  ![prvr1](../../images/prvr1.PNG)
  
  #### K8s Reference Docs
  - https://kubernetes.io/docs/concepts/containers/images/

---

# Images

On déploie de nombreux pods qui hostent différent type d'applications.

Prenons le pod definition file ci-dessous.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

Regardons plus en détails la ligne `image: nginx`.

Le nom est "nginx".

But what is this image and where is this image pulled from?

This name follows Dockers image naming convention nginx.

nginx here is the image or the repository name.

When you say nginx, it's actually library slash nginx.

The first part stands for the user or the account name.

So if you don't provide a user or account name, it assumes it to be library.

Library is the name of the default account where Docker's official images are stored.

These images promote best practices and are maintained by a dedicated team, who are responsible for

reviewing and publishing these official images.

Now, if you were to create your own account and create your own repositories or images under it, then

you would use a similar pattern.

So instead of library it would be your name or your company's name.

Now where are these images stored and pulled from?

Since we have not specified the location where these images are to be pulled from, it is assumed to

be Dockers default registry. docker.io, the docker hub. The DNS name for which is Docker.io.

The registry is where all the images are stored.

Whenever you create a new image or update an image, you push it to the registry and every time anyone

deploys this application, it is pulled from the registry.

```
+----------------------------------------+
|  image: docker.io/library/nginx        |
+----------------------------------------+
          --------- ------- -----
              |        |      |
              o        |      o
           Registry    |    Image/
                       |  Repository
                       o
                  User/Account
```

There are many other popular registries as well.

Google: gcr.io

These are all publicly accessible images that anyone can download and access.

When you have applications built in-house that shouldn't be made available to the public.

Hosting an internal private registry may be a good solution.

Many cloud service providers, such as AWS, Azure, or GCP, provide a private registry by default

on any of these solutions, be it on Docker Hub or Google's Registry or your internal private registry.

You may choose to make a repository private so that it can be accessed using a set of credentials from

Dockers perspective.

To run a container using a private image, you first login to your private registry using a Docker login command

```
docker login private-registry.io
```

Once successful, run the application using the image from the private registry.

```
docker run private-registry.io/apps/internal-app
```

Going back to our Pod definition file to use an image from our private registry, we replace the image

name with the full path to the one in the private registry.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: private-registry.io/apps/internal-app
```

But how do we implement the authentication?

The login part?

How does Kubernetes get the credentials to access the private registry within Kubernetes?

We know that the images are pulled and run by the Docker runtime on the worker nodes (or an other CRE).

How do you pass the credentials to the Docker runtimes on the worker nodes?

For that, we first create a secret object with the credentials in it.

The secret is of type docker registry and we name it regcred.

```
kubectl create secret docker-registry regcred \
   --docker-server=                           \
   --docker-username=                         \
   --docker-password=                         \
   --docker-email=
```

Docker registry is a built in secret type that was built for storing Docker credentials.

We then specify the registry server name, the username to access the registry, the password, and

the email address of the user.

We then specify the secret inside our Pod definition file under the Image Pull Secrets section.

When the Pod is created, Kubernetes or the Kubelets on the worker node uses the credentials from the

secret to pull images.

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: private-registry.io/apps/internal-app
  imagePullSecrets:
  - name: regcred
```
