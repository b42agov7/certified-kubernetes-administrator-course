# KubeConfig 
  - Take me to [Video Tutorial](https://kodekloud.com/topic/kubeconfig/)

In this section, we will take a look at kubeconfig in kubernetes


#### Client uses the certificate file and key to query the kubernetes Rest API for a list of pods using curl.
- You can specify the same using kubectl

  ![kc1](../../images/kc1.PNG)
  
- We can move these information to a configuration file called kubeconfig. And the specify this file as the kubeconfig option in the command.
  ```
  $ kubectl get pods --kubeconfig config
  ```
  
## Kubeconfig File
- The kubeconfig file has 3 sections
  - Clusters
  - Contexts
  - USers
 
  Understanding the kubeconfig File

The kubeconfig file consists of three main sections: clusters, users, and contexts.

    Clusters: Defines the various Kubernetes clusters you need to access. This includes clusters for different environments (development, testing, production), different organizations, or various cloud providers.

    Users: Specifies the user accounts used to access these clusters (e.g., an administrator, a developer, or a service account). These users may hold different privileges across different clusters.

    Contexts: Combines a cluster and a user to define how you connect. A context links a specific user account to a specific cluster.

        Example: You can create a context named admin-at-production that uses the admin account to access the production cluster.

        Example: You might access a Google Cloud-hosted cluster using a developer's credentials to test a deployment.

    Important Note: You are not creating new users or configuring authorization within the cluster itself. Instead, you are using existing users with their current privileges and simply defining which user credentials to use for which cluster.

Ultimately, this configuration saves you from having to repeatedly specify server addresses and user certificates in every kubectl command you run.
  
  ![kc4](../../images/kc4.PNG)
  
  ![kc5](../../images/kc5.PNG)
  
- To view the current file being used
  ```
  $ kubectl config view
  ```
- You can specify the kubeconfig file with kubectl config view with "--kubeconfig" flag
  ```
  $ kubectl config veiw --kubeconfig=my-custom-config
  ```
  
  ![kc6](../../images/kc6.PNG)
  
- How do you update your current context? Or change the current context
  ```
  $ kubectl config use-context <context-name>
  ex: 
  $ kubectl config use-context prod-user@production
  ```
  
  ![kc7](../../images/kc7.PNG)
  
- kubectl config help
  ```
  $ kubectl config -h
  ```
  
  ![kc8](../../images/kc8.PNG)
  
## What about namespaces?

  ![kc9](../../images/kc9.PNG)
 
## Certificates in kubeconfig

  ![kc10](../../images/kc10.PNG)
 
  ![kc12](../../images/kc12.PNG)
  
  ![kc11](../../images/kc11.PNG)
 
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#config


### Bonus 

Kubernetes kubeconfig Guide

A kubeconfig file is a YAML-formatted configuration file used by kubectl (and other Kubernetes clients) to organize and access multiple clusters, users, and authentication mechanisms.

By default, kubectl looks for this file in the user's home directory under .kube/config (~/.kube/config). You can also explicitly specify a custom file using the --kubeconfig flag.
1. Structure of a Kubeconfig File

A standard kubeconfig file contains three core arrays (sections) and a pointer to the active context:
YAML

apiVersion: v1
kind: Config
clusters: []
contexts: []
users: []
current-context: ""

The Three Main Sections

    clusters: Defines the cluster endpoint details, including the server URL and its Certificate Authority (CA).

    users: Defines the user credentials (such as client certificates, tokens, or credentials) used to authenticate against the cluster.

    contexts: Links a specific cluster and a user together under a single friendly name, optionally binding them to a default namespace.

    current-context: Specifies which context kubectl should use by default when executing commands.

2. Example kubeconfig File

Here is an example containing multiple clusters, users, and contexts, with one set as the current active context. It also demonstrates both file path references and base64-encoded certificate data.
YAML

apiVersion: v1
kind: Config
preferences: {}

clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://my-cube-playground.k8s.local:6443
  name: my-cube-playground

- cluster:
    certificate-authority-data: LS0tLS1CRUdJTi... # Base64 encoded CA cert
    server: https://production-cluster.k8s.local:6443
  name: production

users:
- user:
    client-certificate: /etc/kubernetes/pki/cube-admin.crt
    client-key: /etc/kubernetes/pki/cube-admin.key
  name: cube-admin

- user:
    client-certificate-data: LS0tLS1CRUdJTi... # Base64 encoded client cert
    client-key-data: LS0tLS1CRUdJTi...         # Base64 encoded client key
  name: prod-user

contexts:
- context:
    cluster: my-cube-playground
    namespace: development
    user: cube-admin
  name: my-cube-admin@my-cube-playground

- context:
    cluster: production
    namespace: default
    user: prod-user
  name: prod-user@production

current-context: my-cube-admin@my-cube-playground

3. Configuring Namespaces in Contexts

Each cluster can host multiple namespaces. Instead of manually passing the -n <namespace> flag with every command, you can bind a context directly to a specific namespace by adding the namespace field under the context entry:
YAML

contexts:
- context:
    cluster: my-cube-playground
    user: cube-admin
    namespace: development  # Automatically targets the 'development' namespace
  name: dev-context

When you switch to this context, kubectl will default to that namespace automatically.
4. Certificates: File Paths vs. Base64 Encoded Data

When referencing certificates and keys in your configuration, you have two options:

    File Paths (Recommended for local dev): Point directly to the file location on disk.
    YAML

certificate-authority: /path/to/ca.crt
client-certificate: /path/to/client.crt
client-key: /path/to/client.key

Base64 Encoded Data (Recommended for portability/CI-CD): Embed the raw file contents directly into the YAML file encoded in base64.
YAML

    certificate-authority-data: LS0tLS1CRUdJTiBSQVM...
    client-certificate-data: LS0tLS1CRUdJTiBSQVM...
    client-key-data: LS0tLS1CRUdJTiBSQVM...

5. Useful kubectl config Commands

You can view and modify your kubeconfig file directly through kubectl without editing the YAML manually:

    View the current configuration:
    Bash

kubectl config view

Switch the active context (e.g., to production):
Bash

kubectl config use-context prod-user@production

Use a custom kubeconfig file:
Bash

kubectl get pods --kubeconfig /path/to/custom-kubeconfig.yaml
