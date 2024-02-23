## Exercise

1. List all the events in the cluster

2. List the same events sorted by timestamp

3. Create a pod named *www* based on nginx

4. List the events related to this Pod only

5. List all the events related to the *Created* reason

6. List all the events related to a type different from *Normal*

## Documentation

[https://kubernetes.io/docs/reference/kubectl/generated/kubectl_events/](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_events/)

<details>
  <summary markdown="span">Solution</summary>

1. List all the events in the cluster

```
k get events -A
```

2. List the same events sorted by timestamp

```
k get events --sort-by={.metadata.creationTimestamp}
```

3. Create a pod named *www* based on nginx

```
k run www --image=nginx:1.24
```

4. List the events related to this Pod only

kubectl get events --field-selector involvedObject.name=www

5. List all the events related to the *Created* reason

```
kubectl get events --field-selector reason=Created
```

6. List all the events related to a type different from *Normal*

```
kubectl get events --field-selector type=Normal
```

</details>

