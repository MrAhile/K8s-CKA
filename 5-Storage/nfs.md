:warning: This exercise requires a specific configuration and cannot be performed outside of supervised training.

## Exercise

A NFS server is available on *share.techwhale.io*, it exports */nfs-export*

Note: this is a temporary file server widly opened

1. Verify the NFS server is reachable from your worker nodes

Note: you can use the command `showmount -e share.techwhale.io`

2. Install the NFS Provisioner with Helm

Installation instruction can be found at [https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

Warning: make sure to use the name of the server (*share.techwhale.io*) and the exported path (*/nfs-export*) when installing the provisioner 

3. Make sure the provisoner is running fine and a StorageClass has been created

4. Create a PersistentVolumeClaim requesting 100Mi of storage and referencing this StorageClass

Note: you can specify the *ReadWriteMany* accessMode

5. Make sure a PersistentVolume has been created and it is bound to the PVC

6. Create a pod based on alpine:3.15 which mount the content of the PV in /tmp/share. Make sure the pod writes your firstname and the date in /tmp/share/index.html every couple of minutes

Note: the container can use a command similar to "while true; do echo hello from YOUR_FIRSTNAME at $(date) >> /tmp/share/index.html; sleep 100000; done". Make sure to replace *YOUR_FIRSTNAME* with your firstname :)

7. Check the content of /tmp/share/index.html inside the pod's container

8. Delete the pod, the pvc and uninstall the NFS provisioner.

## Documentation

[https://kubernetes.io/docs/concepts/storage/storage-classes/](https://kubernetes.io/docs/concepts/storage/storage-classes/)

<details>
  <summary markdown="span">Solution</summary>

1. Verify the NFS server is reachable from your worker nodes

You will get the same result from worker1 and worker2

```
showmount -e share.techwhale.io
Export list for share.techwhale.io:
/nfs-export 194.182.168.0/22,91.92.118.0/23,91.92.116.0/23,89.145.160.0/22
```

2. Install the NFS Provisioner with Helm

Add the Helm repo containing the NFS provisioner:

```
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```

Install the NFS provisioner providing the path towards the NFS server and the name of the export:

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=share.techwhale.io --set nfs.path=/nfs-export
```

3. Make sure the provisoner is running fine and a StorageClass has been created

Making sure the provisioner is running fine:

```
k get po -l app=nfs-subdir-external-provisioner
```

The *nfs-client* storage class has been created:

```
k get sc
NAME         PROVISIONER                                     RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-client   cluster.local/nfs-subdir-external-provisioner   Delete          Immediate           true                   10s
```

4. Create a PersistentVolumeClaim requesting 100Mi of storage and referencing this StorageClass

Creation of the PVC:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: share
spec: 
  storageClassName: "nfs-client"
  accessModes:
    - ReadWriteMany
  resources:
    requests: 
      storage: 100Mi
EOF
```

5. Make sure a PersistentVolume has been created and it is bound to the PVC

The PVC is bound to a newly created PV:

```
k get pvc,pv
NAME                          STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/share   Bound    pvc-ed1f9e73-285b-49f1-8ae8-c33d1ba96fee   100Mi      RWX            nfs-client     4s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM           STORAGECLASS   REASON   AGE
persistentvolume/pvc-ed1f9e73-285b-49f1-8ae8-c33d1ba96fee   100Mi      RWX            Delete           Bound      default/share   nfs-client              4s
```

6. Create a pod based on alpine:3.15 which mount the content of the PV in /tmp/share. Make sure the pod writes your firstname and the date in /tmp/share/index.html every couple of minutes

The pod can have a specification like the following one:

```
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - image: alpine:3.15
    name: alpine
    command:
    - "/bin/sh"
    - "-c"
    - "while true; do echo hello from YOUR_FIRSTNAME at $(date) >> /tmp/share/index.html; sleep 3600; done"
    volumeMounts:
    - name: share
      mountPath: /tmp/share
  volumes:
  - name: share
    persistentVolumeClaim:
      claimName: share
```

7. Check the content of /tmp/share/index.html inside the pod's container

```
k exec -ti hello -- cat /tmp/share/index.html
hello from YOUR_FIRSTNAME at Fri Apr 1 13:54:34 UTC 2022
```

8. Delete the pod, the pvc and uninstall the NFS provisioner.

Deletion of the pod and the pvc

```
k delete po/hello pvc/share
```

Uninstall of the NFS provisioner

```
helm uninstall nfs-subdir-external-provisioner
```

</details>

