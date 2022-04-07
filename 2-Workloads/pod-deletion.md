## Exercise

1. Create a Pod named *debug* with one container based on *alpine:3.15* executing the command `sleep 10000`. Make sure the pod is running fine

2. Measure the time needed to delete the pod. How to do explain that ?

3. Which command could be used to delete the pod right away ?

<details>
  <summary markdown="span">Solution</summary>

1. Create a Pod named *debug* with one container based on *alpine:3.15* and make sure it executes the command `sleep 10000`

Creation of the pod:

```
k run debug --image=alpine:3.15 --command sleep 10000
```

Making sure the pod is running:

```
k get po debug
NAME    READY   STATUS    RESTARTS   AGE
debug   1/1     Running   0          4s
```

2. Measure the time needed to delete the pod. How to do explain that ?

It takes around 30 seconds for the pod to be deleted:

```
time k delete po debug
pod "debug" deleted

real	0m30.682s  <- you've been waiting more than 30 seconds for the pod to the deleted
user	0m0.096s
sys	  0m0.037s
```

There are some cases where the sigterm signal is not forwarded to the container. In that case, a sigkill is sent after 30 seconds.

3. Which command could be used to delete the pod right away ?

You can force the pod deletion with the following options:

```
k delete po debug --force --grace-period=0
```

</details>

