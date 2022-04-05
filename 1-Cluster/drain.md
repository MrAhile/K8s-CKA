## Exercise

1. Create a deployment named *www*, with 4 replicas of pods based on the nginx:1.20 image

2. Where are the pods scheduled ?

3. Drain the worker1 node. What happened ?

4. Uncordon worker1

5. What happened for the pods belonging to the www deployment ?

6. Force the recreation of the pods. What happened ?

7. Delete the deployment

<details>
  <summary markdown="span">Solution</summary>

1. Create a deployment with 6 replicas of pods based on the nginx:1.20 image

```
k create deploy www --image=nginx:1.20 --replicas=4
```

2. Where are the pods scheduled ?

The pods are splitted between worker1 and worker2 (because master node has a NoScheduled taint that the pods do not tolerate)

```
k get po -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
www-644dfdf68b-6mk2w   1/1     Running   0          33s   10.38.0.1   worker2   <none>           <none>
www-644dfdf68b-crdjl   1/1     Running   0          33s   10.32.0.6   worker1   <none>           <none>
www-644dfdf68b-nhm7b   1/1     Running   0          33s   10.32.0.2   worker1   <none>           <none>
www-644dfdf68b-tsw84   1/1     Running   0          33s   10.38.0.5   worker2   <none>           <none>
```

3. Drain the worker1 node. What happened ?

```
k drain worker1 --ignore-daemonsets
```

The application pods have been evicted from worker1 and are now all running on worker2

```
NAME                   READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
www-644dfdf68b-6mk2w   1/1     Running   0          56s   10.38.0.1   worker2   <none>           <none>
www-644dfdf68b-slvk9   1/1     Running   0          18s   10.38.0.6   worker2   <none>           <none>
www-644dfdf68b-tsw84   1/1     Running   0          57m   10.38.0.5   worker2   <none>           <none>
www-644dfdf68b-xrhhm   1/1     Running   0          9ss   10.38.0.2   worker2   <none>           <none>
```

4. Uncordon worker1

Undordoning a node makes able to receive pods

```
k uncordon worker1
```

5. What happened for the pods belonging to the www deployment ?

Nothing changed regarding the pods belonging to the www deployment:

6. Force the re-creation of the pods

Restarting the deployment

```
k rollout restart deploy/www
```

Forcing the deployment to be restarted will terminate the running pods and create new ones

```
k get po -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
www-b44d96857-bpxr8   1/1     Running   0          5s    10.38.0.5   worker2   <none>           <none>
www-b44d96857-lwx65   1/1     Running   0          5s    10.32.0.3   worker1   <none>           <none>
www-b44d96857-rdv69   1/1     Running   0          6s    10.38.0.2   worker2   <none>           <none>
www-b44d96857-xbssr   1/1     Running   0          6s    10.32.0.2   worker1   <none>           <none>
```

Pods are now distrobuted on the worker1 and worker2 nodes

7. Delete the deployment

```
k delete deploy www
```
</details>

