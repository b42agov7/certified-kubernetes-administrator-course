### Admission Controllers

As we know, when we want to create a pod—for instance, using `kubectl`—the request is sent to the `kube-api-server`. Before the request is successfully processed by the `kube-api-server`, it must go through several stages.

The first stage is **authentication**.

When a command is executed via `kubectl`, the authentication process is carried out using the certificate contained in the configuration file.

Following the authentication stage, the request proceeds to an **authorization** stage. It is during this phase that it is verified whether or not the users are permitted to perform the intended operations.

Authorizations are implemented using RBAC.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "get", "create", "update", "delete"] 
```

This role grants full rights to manage pods (create, read, modify, delete) within a single namespace. It provides absolutely no access to other resources (services, secrets, etc.).

If the request complies with the specifications defined in the RBAC, it proceeds to the subsequent stages; otherwise, it is rejected.
Roles created via RBAC operate at the level of the Kubernetes API. This involves defining what a user is authorized to access and the types of operations they are permitted to perform in relation to the API.

What if you want to go beyond simply defining a user's access level to an object?

For example, you might want to prohibit public images, avoid using "latest" versions, permit or restrict specific capabilities, or enforce the use of labels. These actions can be carried out using **Admission Controllers**.

Admission Controllers allow you to monitor and control activity and usage within the cluster.

They make it possible to modify an initial request or perform additional operations before a pod is created.

There are a number of admission controllers pre-integrated into Kubernetes.

For example:
- `AlwaysPullImages`, which ensures that images are always pulled whenever a pod is created.
- `DefaultStorageClass`, which monitors the creation of PVCs (PersistentVolumeClaims) and automatically assigns the default storage class if one is not specified.
- `EventRateLimit`, which allows for setting a limit on the number of requests the API server can process at once, preventing it from being overwhelmed by requests.
- `NameSpaceExists`, which rejects requests that reference a namespace that does not exist.

There are admission controllers that are not enabled by default. Here are a few examples.
`NamespaceAutoProvision` allows for the creation of namespaces if they do not already exist.

To list the admission controllers enabled by default:
```
kube-apiserver -h | grep "enable-admission-plugins"
```

With kubeadm setup:
```
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```

To enable Admission Controllers, depending on the installation, you must modify the `--enable-admission-plugins` section.

*No kubeadm*

Edit kube-apiserver.service
```
ExecStart=/usr/local/bin/kube-apiserver \\
  -- ...
  --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
  -- ...
```

*kubeadm*

Edit /etc/kubernetes/manifest/kube-apiserver.yaml
```
apiVersion: v1
...
spec:
  containers:
  - command:
    - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
...
```

To disable Admission Controllers, depending on the installation, you must modify the `--disable-admission-plugins` section.

*No kubeadm*

Edit kube-apiserver.service
```
ExecStart=/usr/local/bin/kube-apiserver \\
  -- ...
  --disable-admission-plugins=NodeRestriction,DefaultStorageClass
  -- ...
```

*kubeadm*

Edit /etc/kubernetes/manifest/kube-apiserver.yaml
```
apiVersion: v1
...
spec:
  containers:
  - command:
    - --disable-admission-plugins=DefaultStorageClass
...
```

Note that the NameSpaceAutoProvision and NameSpaceExists admission controllers are now obsolete and have been replaced by the Namespace Lifecycle admission controllers.
