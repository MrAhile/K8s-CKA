## Exercise

1. Create a pod named *secure* with a single container based on the *alpine:3.15* image running the command "sleep 3600" with the user ID 1000 and the group ID 1000

2. Run a shell in the container verify the owner of the process

3. Delete the Pod

## Documentation

[https://kubernetes.io/docs/tasks/configure-pod-container/security-context/](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

<details>
  <summary markdown="span">Solution</summary>

1. Create a pod called secure with the alpine image. The container should run the command "sleep 3600" with the user ID 1000 and the group ID 1000

```
apiVersion: v1
kind: Pod
metadata:
  name: secure
spec:
  containers:
  - image: alpine:3.15
    name: pod
    command:
    - "sleep"
    - "3600"
    securityContext:
      runAsUser: 1000
      runAsGroup: 1000
```

2. Run a shell in the container verify the process' owner

```
k exec secure -- ps aux | grep sleep
1 1000      0:00 sleep 3600
```

3. Delete the Pod

```
k delete po secure
```
</details>

