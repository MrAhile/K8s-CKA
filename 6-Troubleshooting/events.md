## Exercise

1. List all the events in the cluster

2. List the same events sorted by timestamp

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

</details>

