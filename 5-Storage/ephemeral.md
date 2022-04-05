
## Exercise

1. Create a pod with a single container based on *mongo:5.0* and make it persist its data in a volume linked to the pod's lifecycle

Hint: Mongo persists the data in it's */data/db* folder

2. Run a shell in the container and list the content of the */data/db* folder

3. Delete the pod

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

3. Delete the pod

```
k delete po mongo
```
</details>

