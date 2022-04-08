
## Exercise

1. Create a pod with a single container based on *mongo:5.0* and make it persist its data in a volume linked to the pod's lifecycle

Hint: Mongo persists the data in it's */data/db* folder

2. Run a shell in the container and list the content of the */data/db* folder

3. Get the unique identifier of the pod

Hint: this can be retrieved in .metadata.uid

4. On which node is this pod running

5. Run a shell on the node this pod is running on and retreive the content of the emtpyDir volume

Hint: it's located in */var/lib/kubelet/pods/POD_ID/volumes/kubernetes.io~empty-dir/data*

6. Delete the pod

## Documentation

[https://kubernetes.io/docs/concepts/storage/volumes/#emptydir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir)

<details>
  <summary markdown="span">Solution</summary>

1. Create a pod with a single container based on *mongo:5.0* and make it persist its data in a volume linked to the pod's lifecycle

```
kubectl run mongo --image=mongo:5.0 --dry-run=client -o yaml > pod.yaml
```

Add the volume section:

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mongo
  name: mongo
spec:
  containers:
  - image: mongo:5.0
    name: mongo
    volumeMounts:
    - name: data
      mountPath: /data/db
  volumes:
  - name: data
    emptyDir: {}
```

Create the pod:

```
k apply -f pod.yaml
```

2. Run a shell in the container and list the content of the */data/db* folder

```
k exec mongo -- ls /data/db
```

3. Get the unique identifier of the pod

```
k get po mongo -o jsonpath={.metadata.uid}
```

4. On which node is this pod running

You can get the information with:

```
k get po mongo -o wide
```

5. Run a shell on the node this pod is running on and retreive the content of the emtpyDir volume

From the node the pod is running on, list the content of the following folder making sure to replace POD_ID with the unique identifier you get in question 3.

```
/var/lib/kubelet/pods/POD_ID/volumes/kubernetes.io~empty-dir/data
```

6. Delete the pod

```
k delete po mongo
```
</details>

