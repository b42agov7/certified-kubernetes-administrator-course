# Backup and Restore Methods
  - Take me to [Video Tutorial](https://kodekloud.com/topic/backup-and-restore-methods/)
  
In this section, we will take a look at backup and restore methods

## Backup Candidates
 
 ![bc](../../images/bc.PNG)
 
## Resource Configuration
- Imperative way
  
  ![rci](../../images/rci.PNG)

- Declarative Way (Preferred approach)
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: myapp-pod
    labels:
      app: myapp
      type: front-end
  spec:
    containers:
    - name: nginx-container
      image: nginx
  ```
 ![rcd](../../images/rcd.PNG)
 
- A good practice is to store resource configurations on source code repositories like github.

  ![rcd1](../../images/rcd1.PNG)

## Backup - Resource Configs

  ```
  $ kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml (only for few resource groups)
  ```

- There are many other resource groups that must be considered. There are tools like **`ARK`** or now called **`Velero`** by Heptio that can do this for you.

  ![brc](../../images/brc.PNG)
  
## Backup - ETCD

The ETCD cluster stores information about the state of our cluster.

So information about the cluster itself, the nodes and every other resources created within the cluster are stored here.

As we have seen, the etcd cluster is hosted on the master nodes.

While configuring etcd, we specified a location where all the data would be stored. It's `--data-dir=/var/lib/etcd`

- So, instead of backing up resources as before, you may choose to backup the ETCD cluster itself. 
  
  ![be](../../images/be.PNG)
  
- You can take a snapshot of the etcd database by using **`etcdctl`** utility snapshot save command.
  ```
  $ ETCDCTL_API=3 etcdctl snapshot save snapshot.db
  ```

  You can view the status of the backup using the snapshot status command.
  ```
  $  ETCDCTL_API=3 etcdctl snapshot status snapshot.db
  ```
  ![be1](../../images/be1.PNG)
  
## Restore - ETCD

- To restore etcd from the backup at later in time. First stop kube-apiserver service
  ```
  $ sysemctl stop kube-apiserver 
  ```
- Run the etcdctl snapshot restore command
```

```

- Update the etcd service (--data-dir)
- Reload system configs
  ```
  $ systemctl daemon-reload
  ```
- Restart etcd
  ```
  $ systemctl restart etcd
  ```
  
  ![er](../../images/er.PNG)
  
- Start the kube-apiserver
  ```
  $ systemctl start kube-apiserver 
  ```
#### With all etcdctl commands specify the cert,key,cacert and endpoint for authentication.
```
$ ETCDCTL_API=3 etcdctl \
  snapshot save /tmp/snapshot.db \
  --endpoints=https://[127.0.0.1]:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/etcd-server.crt \
  --key=/etc/kubernetes/pki/etcd/etcd-server.key
```

  ![erest](../../images/erest.PNG)
  
#### K8s Reference Docs
- https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/


 





----





controlplane ~ ✖ ls /etc/kubernetes/pki/etcd/
ca.crt  ca.key  healthcheck-client.crt  healthcheck-client.key  peer.crt  peer.key  server.crt  server.key

controlplane ~ ➜  ETCDCTL_API=3 etcdctl snapshot save /opt/snapshot-pre-boot.db --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/serv
er.key
{"level":"warn","ts":"2026-11-04T17:30:42.480702Z","caller":"flags/flag.go:94","msg":"unrecognized environment variable","environment-variable":"ETCDCTL_API=3"}
{"level":"info","ts":"2026-11-04T17:30:42.490920Z","caller":"snapshot/v3_snapshot.go:83","msg":"created temporary db file","path":"/opt/snapshot-pre-boot.db.part"}
{"level":"info","ts":"2026-11-04T17:30:42.495347Z","logger":"client","caller":"v3@v3.6.6/maintenance.go:236","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":"2026-11-04T17:30:42.505541Z","caller":"snapshot/v3_snapshot.go:96","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":"2026-11-04T17:30:42.513120Z","logger":"client","caller":"v3@v3.6.6/maintenance.go:302","msg":"completed snapshot read; closing"}
{"level":"info","ts":"2026-11-04T17:30:42.525521Z","caller":"snapshot/v3_snapshot.go:111","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"1.8 MB","took":"34.562998ms","etcd-version":"3.6.0"}
{"level":"info","ts":"2026-11-04T17:30:42.527201Z","caller":"snapshot/v3_snapshot.go:121","msg":"saved","path":"/opt/snapshot-pre-boot.db"}
Snapshot saved at /opt/snapshot-pre-boot.db
Server version 3.6.0

controlplane ~ ➜  

controlplane ~ ➜  etcdutl snapshot status /opt/snapshot-pre-boot.db --write-out table
+----------+----------+------------+------------+---------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE | VERSION |
+----------+----------+------------+------------+---------+
| f40cef5c |     1308 |        392 |     1.8 MB |   3.6.0 |
+----------+----------+------------+------------+---------+

controlplane ~ ➜  

---

ETCD -> static pod

controlplane ~ ➜  crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD                                    NAMESPACE
d5019d804825a       ead0a4a53df89       36 minutes ago      Running             coredns                   0                   077ed077a10fe       coredns-6f6c7df987-jzn9c               kube-system
32090fb38cb8f       ead0a4a53df89       36 minutes ago      Running             coredns                   0                   3dceaae6db1eb       coredns-6f6c7df987-s9dvx               kube-system
4dc78fc250ed0       01cdfa8dd262f       36 minutes ago      Running             kube-flannel              0                   3a5b715e26b91       kube-flannel-ds-rtzqf                  kube-flannel
9f3e6d0aea4d0       01cdfa8dd262f       36 minutes ago      Exited              install-cni               0                   3a5b715e26b91       kube-flannel-ds-rtzqf                  kube-flannel
02a17c439cb57       a55d1bad692b7       36 minutes ago      Exited              install-cni-plugin        0                   3a5b715e26b91       kube-flannel-ds-rtzqf                  kube-flannel
66c74087eb568       32652ff1bbe6b       36 minutes ago      Running             kube-proxy                0                   67f3869ead206       kube-proxy-bqpz8                       kube-system
19684614416a7       2c9a4b058bd7e       36 minutes ago      Running             kube-controller-manager   0                   2432cc30f86c4       kube-controller-manager-controlplane   kube-system
ef909d195492a       5c6acd67e9cd1       36 minutes ago      Running             kube-apiserver            0                   b8ec0df9ffd13       kube-apiserver-controlplane            kube-system
69a6b521bae91       550794e3b12ac       36 minutes ago      Running             kube-scheduler            0                   7c2b31b1d547f       kube-scheduler-controlplane            kube-system
8a43b59406ea1       0a108f7189562       36 minutes ago      Running             etcd                      0                   1509510e924e6       etcd-controlplane                      kube-system

controlplane ~ ➜ 

sudo mkdir -p /tmp/k8s-backup

controlplane ~ ➜  sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/k8s-backup/

controlplane ~ ➜  ls /tmp/k8s-backup/
kube-apiserver.yaml

controlplane ~ ➜  crictl ps -a
CONTAINER           IMAGE               CREATED                  STATE               NAME                      ATTEMPT             POD ID              POD                                    NAMESPACE
02f036ba07c08       2c9a4b058bd7e       Less than a second ago   Running             kube-controller-manager   1                   2432cc30f86c4       kube-controller-manager-controlplane   kube-system
73d7734ee115c       550794e3b12ac       2 seconds ago            Running             kube-scheduler            1                   7c2b31b1d547f       kube-scheduler-controlplane            kube-system
d5019d804825a       ead0a4a53df89       43 minutes ago           Running             coredns                   0                   077ed077a10fe       coredns-6f6c7df987-jzn9c               kube-system
32090fb38cb8f       ead0a4a53df89       43 minutes ago           Running             coredns                   0                   3dceaae6db1eb       coredns-6f6c7df987-s9dvx               kube-system
4dc78fc250ed0       01cdfa8dd262f       43 minutes ago           Running             kube-flannel              0                   3a5b715e26b91       kube-flannel-ds-rtzqf                  kube-flannel
9f3e6d0aea4d0       01cdfa8dd262f       43 minutes ago           Exited              install-cni               0                   3a5b715e26b91       kube-flannel-ds-rtzqf                  kube-flannel
02a17c439cb57       a55d1bad692b7       43 minutes ago           Exited              install-cni-plugin        0                   3a5b715e26b91       kube-flannel-ds-rtzqf                  kube-flannel
66c74087eb568       32652ff1bbe6b       43 minutes ago           Running             kube-proxy                0                   67f3869ead206       kube-proxy-bqpz8                       kube-system
19684614416a7       2c9a4b058bd7e       43 minutes ago           Exited              kube-controller-manager   0                   2432cc30f86c4       kube-controller-manager-controlplane   kube-system
ef909d195492a       5c6acd67e9cd1       43 minutes ago           Running             kube-apiserver            0                   b8ec0df9ffd13       kube-apiserver-controlplane            kube-system
69a6b521bae91       550794e3b12ac       43 minutes ago           Exited              kube-scheduler            0                   7c2b31b1d547f       kube-scheduler-controlplane            kube-system
8a43b59406ea1       0a108f7189562       43 minutes ago           Running             etcd                      0                   1509510e924e6       etcd-controlplane                      kube-system

controlplane ~ ➜  



crictl ps | grep kube-apiserver



controlplane ~ ➜  ls /var/lib/etcd/
member

controlplane ~ ➜  etcdutl snapshot restore /opt/snapshot-pre-boot.db --data-dir=/var/lib/etcd-backup
2026-11-04T17:48:54Z    info    snapshot/v3_snapshot.go:305     restoring snapshot      {"path": "/opt/snapshot-pre-boot.db", "wal-dir": "/var/lib/etcd-backup/member/wal", "data-dir": "/var/lib/etcd-backup", "snap-dir": "/var/lib/etcd-backup/member/snap", "initial-memory-map-size": 10737418240}
2026-11-04T17:48:54Z    info    bbolt   backend/backend.go:203  Opening db file (/var/lib/etcd-backup/member/snap/db) with mode -rw------- and with options: {Timeout: 0s, NoGrowSync: false, NoFreelistSync: true, PreLoadFreelist: false, FreelistType: , ReadOnly: false, MmapFlags: 8000, InitialMmapSize: 10737418240, PageSize: 0, NoSync: false, OpenFile: 0x0, Mlock: false, Logger: 0xc0003861c8}
2026-11-04T17:48:54Z    info    bbolt   bbolt@v1.4.3/db.go:321  Opening bbolt db (/var/lib/etcd-backup/member/snap/db) successfully
2026-11-04T17:48:54Z    info    schema/membership.go:138        Trimming membership information from the backend...
2026-11-04T17:48:54Z    info    bbolt   backend/backend.go:203  Opening db file (/var/lib/etcd-backup/member/snap/db) with mode -rw------- and with options: {Timeout: 0s, NoGrowSync: false, NoFreelistSync: true, PreLoadFreelist: false, FreelistType: , ReadOnly: false, MmapFlags: 8000, InitialMmapSize: 10737418240, PageSize: 0, NoSync: false, OpenFile: 0x0, Mlock: false, Logger: 0xc000386220}
2026-11-04T17:48:54Z    info    bbolt   bbolt@v1.4.3/db.go:321  Opening bbolt db (/var/lib/etcd-backup/member/snap/db) successfully
2026-11-04T17:48:54Z    info    membership/cluster.go:424       added member    {"cluster-id": "cdf818194e3a8c32", "local-member-id": "0", "added-peer-id": "8e9e05c52164694d", "added-peer-peer-urls": ["http://localhost:2380"], "added-peer-is-learner": false}
2026-11-04T17:48:54Z    info    bbolt   backend/backend.go:203  Opening db file (/var/lib/etcd-backup/member/snap/db) with mode -rw------- and with options: {Timeout: 0s, NoGrowSync: false, NoFreelistSync: true, PreLoadFreelist: false, FreelistType: , ReadOnly: false, MmapFlags: 8000, InitialMmapSize: 10737418240, PageSize: 0, NoSync: false, OpenFile: 0x0, Mlock: false, Logger: 0xc0003862d8}
2026-11-04T17:48:54Z    info    bbolt   bbolt@v1.4.3/db.go:321  Opening bbolt db (/var/lib/etcd-backup/member/snap/db) successfully
2026-11-04T17:48:54Z    info    snapshot/v3_snapshot.go:333     restored snapshot       {"path": "/opt/snapshot-pre-boot.db", "wal-dir": "/var/lib/etcd-backup/member/wal", "data-dir": "/var/lib/etcd-backup", "snap-dir": "/var/lib/etcd-backup/member/snap", "initial-memory-map-size": 10737418240}

controlplane ~ ➜  


controlplane ~ ➜  ls /var/lib/etcd-backup/
member



controlplane ~ ➜  cat /etc/kubernetes/manifests/etcd.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.244.241.7:2379
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://10.244.241.7:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --feature-gates=InitialCorruptCheck=true
    - --initial-advertise-peer-urls=https://10.244.241.7:2380
    - --initial-cluster=controlplane=https://10.244.241.7:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://10.244.241.7:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://10.244.241.7:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --watch-progress-notify-interval=5s
    image: registry.k8s.io/etcd:3.6.6-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /livez
        port: probe-port
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    ports:
    - containerPort: 2381
      name: probe-port
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTP
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd
      type: DirectoryOrCreate
    name: etcd-data
status: {}

sudo cp /etc/kubernetes/manifests/etcd.yaml /etc/kubernetes/manifests/etcd.yaml.bak


controlplane ~ ➜  diff -u /etc/kubernetes/manifests/etcd.yaml.bak /etc/kubernetes/manifests/etcd.yaml
--- /etc/kubernetes/manifests/etcd.yaml.bak     2026-11-04 17:54:01.895151536 +0000
+++ /etc/kubernetes/manifests/etcd.yaml 2026-11-04 17:56:08.370290026 +0000
@@ -15,7 +15,7 @@
     - --advertise-client-urls=https://10.244.241.7:2379
     - --cert-file=/etc/kubernetes/pki/etcd/server.crt
     - --client-cert-auth=true
-    - --data-dir=/var/lib/etcd
+    - --data-dir=/var/lib/etcd-backup
     - --feature-gates=InitialCorruptCheck=true
     - --initial-advertise-peer-urls=https://10.244.241.7:2380
     - --initial-cluster=controlplane=https://10.244.241.7:2380
@@ -88,7 +88,7 @@
       type: DirectoryOrCreate
     name: etcd-certs
   - hostPath:
-      path: /var/lib/etcd
+      path: /var/lib/etcd-backup
       type: DirectoryOrCreate
     name: etcd-data
 status: {}

controlplane ~ ✖ 


controlplane ~ ➜  crictl ps | grep etcd
db52656ec2731       0a108f7189562       3 minutes ago       Running             etcd                      1                   7165886f1bffe       etcd-controlplane                      kube-system

controlplane ~ ➜  crictl ps | grep etcd
db52656ec2731       0a108f7189562       3 minutes ago       Running             etcd                      1                   7165886f1bffe       etcd-controlplane                      kube-system

controlplane ~ ➜  


controlplane ~ ➜  cat /etc/kubernetes/manifests/etcd.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.244.34.225:2379
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://10.244.34.225:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd-backup
    - --feature-gates=InitialCorruptCheck=true
    - --initial-advertise-peer-urls=https://10.244.34.225:2380
    - --initial-cluster=controlplane=https://10.244.34.225:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://10.244.34.225:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://10.244.34.225:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --watch-progress-notify-interval=5s
    image: registry.k8s.io/etcd:3.6.6-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /livez
        port: probe-port
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    ports:
    - containerPort: 2381
      name: probe-port
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTP
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd-backup
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-backup
      type: DirectoryOrCreate
    name: etcd-data
status: {}

controlplane ~ ➜  




controlplane ~ ➜  cat /etc/kubernetes/manifests/etcd.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.244.34.225:2379
  labels:
    component: etcd
    tier: control-plane
  name: etcd
  namespace: kube-system
spec:
  containers:
  - command:
    - etcd
    - --advertise-client-urls=https://10.244.34.225:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd-backup
    - --feature-gates=InitialCorruptCheck=true
    - --initial-advertise-peer-urls=https://10.244.34.225:2380
    - --initial-cluster=controlplane=https://10.244.34.225:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --listen-client-urls=https://127.0.0.1:2379,https://10.244.34.225:2379
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://10.244.34.225:2380
    - --name=controlplane
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --watch-progress-notify-interval=5s
    image: registry.k8s.io/etcd:3.6.6-0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /livez
        port: probe-port
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: etcd
    ports:
    - containerPort: 2381
      name: probe-port
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTP
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 100m
        memory: 100Mi
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /var/lib/etcd-backup
      name: etcd-data
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-backup
      type: DirectoryOrCreate
    name: etcd-data
status: {}

controlplane ~ ➜  ls /tmp/
etcd-download                   install-etcd.out  k8s-backup           startup-script.sh
etcd-v3.6.6-linux-amd64.tar.gz  install-etcd.sh   kube-apiserver.yaml

controlplane ~ ➜  mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

controlplane ~ ➜  sudo mv /etc/kubernetes/manifests/kube-controller-manager.yaml /tmp/

controlplane ~ ➜  sudo mv /etc/kubernetes/manifests/kube-scheduler.yaml /tmp/

controlplane ~ ➜  sleep 20

controlplane ~ ➜  sudo mv /tmp/kube-controller-manager.yaml /tmp/kube-scheduler.yaml /etc/kubernetes/manifests/

controlplane ~ ➜  sudo systemctl restart kubelet

controlplane ~ ➜  kubectl get deployments,services -n default
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/blue   3/3     3            3           25m
deployment.apps/red    2/2     2            2           25m

NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/blue-service   NodePort    172.20.181.181   <none>        80:30082/TCP   25m
service/kubernetes     ClusterIP   172.20.0.1       <none>        443/TCP        33m
service/red-service    NodePort    172.20.55.247    <none>        80:30080/TCP   25m

controlplane ~ ➜  



Pour résumer clairement avec les notions de hostPath (le dossier physique sur votre serveur) et de mountPath (le chemin virtuel à l'intérieur du conteneur etcd), voici exactement ce qui causait votre erreur :
Ce qui s'est passé :

    Le désaccord d'adresse :
    Dans votre fichier de configuration etcd.yaml, vous aviez configuré le conteneur pour qu'il cherche ses fichiers dans /var/lib/etcd-backup (grâce à l'argument --data-dir). Mais le conteneur montait le dossier physique de l'hôte (hostPath: /var/lib/etcd-backup) sur un chemin virtuel interne (mountPath: /var/lib/etcd) qui ne correspondait plus.

    La conséquence :
    Le conteneur cherchait au mauvais endroit en interne, ce qui fait qu'il démarrait sur une base vide ou corrompue du point de vue de l'API Server. Résultat : l'API Server perdaitt la lecture des données et des rôles de sécurité, ce qui bloquait tout avec l'erreur Forbidden.

    La correction :
    En alignant parfaitement le hostPath (sur la machine) et le mountPath (dans le conteneur) sur /var/lib/etcd-backup, le conteneur a enfin pu traverser la "porte" virtuelle et lire directement le bon contenu de votre sauvegarde restaurée.
    
    
Pour comprendre hostPath et mountPath, imaginez que vous avez un conteneur Kubernetes (comme etcd) qui a besoin d'enregistrer des fichiers importants sur le disque dur de la machine physique pour ne pas les perdre en cas de redémarrage.

Ces deux termes fonctionnent ensemble pour faire une passerelle entre la machine physique (l'hôte) et l'intérieur du conteneur :

    hostPath : C'est l'emplacement réel du dossier sur la machine physique (par exemple, /var/lib/etcd-backup). C'est là que vos fichiers sont stockés en sécurité sur le serveur.

    mountPath : C'est le chemin à l'intérieur du conteneur. C'est la façon dont ce même dossier va apparaître pour l'application qui tourne à l'intérieur (par exemple, /var/lib/etcd).

Pourquoi associer les deux ?

Le conteneur vit dans sa propre bulle isolée et ne voit pas directement le disque dur du serveur. Grâce au mountPath, Kubernetes prend le dossier indiqué par le hostPath sur le serveur et le "branche" virtuellement à l'intérieur du conteneur.

Ainsi, quand l'application écrit des données dans son mountPath, elles sont en réalité enregistrées en toute sécurité dans le hostPath sur le disque de la machine physique.
