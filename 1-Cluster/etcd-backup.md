## Exercise

1. Check API Server to etcd communication

From the controlplane node, check the status of etcd

```
sudo ETCDCTL_API=3 etcdctl \
--endpoints localhost:2379 \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
endpoint health
```

You should get a result similar to:

```
localhost:2379 is healthy: successfully committed proposal: took = 14.891442ms
```

Note: this demo cluster only uses one etcd instance. In a production cluster at least 3 etcd instances are required, these ones could run on the controlplane nodes (stacked etcd) or on external hosts (external etcd)

2. Backup etcd

Before creating an etcd backup, create a simple test deployment:

```
kubectl create deploy nginx --image=nginx:1.20 --replicas=4
```

and make sure the 4 pods are running:

```
k get po -l app=nginx
```

Next create an etcd backup, this generates the file *snapshot.db* in the current folder:

```
sudo ETCDCTL_API=3 etcdctl snapshot save \
--endpoints localhost:2379 \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
snapshot.db
```

Then make sure the backup was done correctly

```
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
```

You should get a result similar to:

```
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| 7bc4b430 |   114953 |       1048 |     3.0 MB |
+----------+----------+------------+------------+
```

3. Restore etcd backup

Before restoring the previous backup, run a new deployment in the cluster:

```
k create deploy mongo --image=mongo:5.0
```

Next make sure the cluster now contains the 2 deployments created above:

```
k get deploy,po
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongo   1/1     1            1           49s
deployment.apps/nginx   4/4     4            4           3m38s

NAME                         READY   STATUS    RESTARTS   AGE
pod/mongo-857d4c74ff-khkbf   1/1     Running   0          49s
pod/nginx-6d777db949-9qg7h   1/1     Running   0          3m38s
pod/nginx-6d777db949-hlb8b   1/1     Running   0          3m38s
pod/nginx-6d777db949-rlzl8   1/1     Running   0          3m38s
pod/nginx-6d777db949-scxt5   1/1     Running   0          3m38s
```

Then restore the backup in the */var/lib/etcd-snapshot* folder (this one will be created automatically in the restoration process):

```
sudo ETCDCTL_API=3 etcdctl snapshot restore \
--endpoints localhost:2379 \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--data-dir /var/lib/etcd-snapshot \
snapshot.db
```

Next, it is necessary to stop the API Server, this can be done by moving the API Server manifests out of the */etc/kubernetest/manifests* folder

```
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/
```

Next it is necessary to modify the configuration of etcd so it take into account the new folder (the one containing the restored backup). Change this path in your etcd manifests (*/etc/kubernetes/manifests/etcd.yaml*), as illustrated in the following example:


```
...
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-snapshot          <- folder to change
      type: DirectoryOrCreate
    name: etcd-data
```

After a few tens of seconds the etcd pods will be back online with the content of the backup.

Restart the API Server moving the specification back in the */etc/kubernetes/manifests* folder

```
sudo mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
```

Then make sure only the nginx deployment exists in the cluster:

```
k get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   4/4     4            4           12m
```

The mongo deployment is not listed as it was created after the etcd backup was created