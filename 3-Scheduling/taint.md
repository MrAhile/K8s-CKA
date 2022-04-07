## Exercise

1. Check the taints of the cluster's nodes. What does that mean ?

2. Create the specification of a Deployment named *nginx* with 6 replicas of Pods based on the nginx:1.20 image, then create the Deployment.

3. Where are the Pods scheduled ?

4. Change the specification of the Deployment so the Pods tolerate the taint present on the master node. Update the Deployment resource.

5. Where are the Pods scheduled ?

6. Delete the deployment

## Documentation

[https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

<details>
  <summary markdown="span">Solution</summary>

1. Check the taints of the cluster's nodes. What does that mean ?

Only the master node as a Taint, this one prevents the application Pod from being scheduled on the master

```
k get nodes master -o jsonpath={.spec.taints}
[{"effect":"NoSchedule","key":"node-role.kubernetes.io/master"}]
```

The NoSchedule taint prevent Pods which do not tolerate the taint to be scheduled on that node

2. Create the specification of a Deployment named *nginx* with 6 replicas of Pods based on the nginx:1.20 image, then create the Deployment.

Specification:

```
k create deploy nginx --replicas 6 --image=nginx:1.20 --dry-run=client -o yaml > deploy.yaml
```

Creation of the Deployment

```
k apply -f deploy.yaml
```

3. Where are the Pods scheduled ?

```
k get po -o wide
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
nginx-6d777db949-2kxhp   1/1     Running   0          8s    10.38.0.5   worker2   <none>           <none>
nginx-6d777db949-6d7ms   1/1     Running   0          8s    10.32.0.7   worker1   <none>           <none>
nginx-6d777db949-8vl6j   1/1     Running   0          8s    10.32.0.6   worker1   <none>           <none>
nginx-6d777db949-rqql4   1/1     Running   0          8s    10.38.0.6   worker2   <none>           <none>
nginx-6d777db949-tfbjb   1/1     Running   0          8s    10.32.0.2   worker1   <none>           <none>
nginx-6d777db949-vs6wl   1/1     Running   0          8s    10.38.0.1   worker2   <none>           <none>
```

The Pods are deployed either on worker1 or on worker2. None are deployed on the master node because of the NoSchedule taint that the Pods do not tolerate.

4. Change the specification of the Deployment so the Pods tolerate the taint present on the master node. Update the Deployment resource.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 6
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.20
        name: nginx
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
```

Update the resource:

```
k apply -f deploy.yaml
```

5. Where are the Pods scheduled ?

Due to the toleration of the taint, Pods can now be scheduled on master as well

```
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE      NOMINATED NODE   READINESS GATES
nginx-7b788fb97d-2vrv4   1/1     Running   0          59s   10.32.0.8   worker1   <none>           <none>
nginx-7b788fb97d-bjlvk   1/1     Running   0          57s   10.40.0.2   master    <none>           <none>
nginx-7b788fb97d-gdn6x   1/1     Running   0          59s   10.38.0.1   worker2   <none>           <none>
nginx-7b788fb97d-qkp78   1/1     Running   0          59s   10.40.0.1   master    <none>           <none>
nginx-7b788fb97d-rb74s   1/1     Running   0          57s   10.32.0.6   worker1   <none>           <none>
nginx-7b788fb97d-tj7f9   1/1     Running   0          57s   10.38.0.7   worker2   <none>           <none>
```

6. Delete the deployment

```
k delete deploy/nginx
```

</details>

