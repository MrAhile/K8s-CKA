## Exercise

1. Create a namespace named *dev*

2. Create a resource quota named *limit-pod-number* limiting the number of pods to 5 in that namespace

3. Create a deployment named *ghost* with 10 replicas based on ghost:4 in the namespace *dev*

4. What do you observe ?

5. Delete the deployment, the quota and the namespace

## Documentation

[https://kubernetes.io/docs/concepts/policy/resource-quotas/](https://kubernetes.io/docs/concepts/policy/resource-quotas/)

<details>
  <summary markdown="span">Solution</summary>

1. Create a namespace named *dev*

```
k create ns dev
```

2. Create a resource quota named *limit-pod-number* limiting the number of pods to 5 in that namespace

```
k -n dev create quota limit-pod-number --hard=pods=5
```

3. Create a deployment named *ghost* with 10 replicas based on ghost:4 in the namespace *dev*

```
k -n dev create deploy ghost --image=ghost:4 --replicas=10
```

4. What do you observe ?

Only 5 of the 10 replicas are running, the other ones cannot be created in the namespace because of the limitation

```
k -n dev get po -l app=ghost
NAME                     READY   STATUS    RESTARTS   AGE
ghost-5d77b859d5-222q2   1/1     Running   0          26s
ghost-5d77b859d5-96z94   1/1     Running   0          26s
ghost-5d77b859d5-hlbbj   1/1     Running   0          26s
ghost-5d77b859d5-kbdf4   1/1     Running   0          26s
ghost-5d77b859d5-w8rfm   1/1     Running   0          26s
```

5. Delete the deployment, the quota and the namespace

```
k -n dev delete deploy/ghost quota/limit-pod-number 
k delete ns dev
```

</details>

