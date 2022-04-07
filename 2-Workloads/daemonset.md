## Exercise

1. Create the specification of a DaemonSet named *log*. Each pod must be based on the alpine:3.15 image they must be configured to read the log files located in */var/log/pods* on the node and stream them on their stdout

Note: path of the log files to stream is /var/log/pods/\*/\*/\*.log

2. Create the DaemonSet. Where are the pods scheduled ? 

3. Change the specification so there is a pod on each node of the cluster

4. Verify the pod can stream the node's log

5. Delete the DaemonSet

## Documentation

[https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

<details>
  <summary markdown="span">Solution</summary>

1. Create the specification of a DaemonSet named *log*. Each pod must be based on the alpine:3.15 image they must be configured to read the log files located in */var/log/pods* on the node and stream them on their stdout

As it is not possible to create a DaemonSet directly with kubectl, we start by creating a Deployment

```
k create deploy log --image=alpine:3.15 --dry-run=client -o yaml > spec.yaml
```

Then we modify the Deployment to change it into a DaemonSet:
- modification of the *kind* from Deployment to DaemonSet
- removal of the *replicas* and *strategy* properties

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      containers:
      - image: alpine:3.15
        name: alpine
```

Next we add a volume which gives access to the */var/log/pods* folder of the node

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      containers:
      - image: alpine:3.15
        name: alpine
      volumes:
      - name: varlogpods
        hostPath:
          path: /var/log/pods
```

Then we mount the volume into the filesystem of the alpine container in */var/log/pods*

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      containers:
      - image: alpine:3.15
        name: alpine
        volumeMounts:
        - name: varlogpods
          mountPath: /var/log/pods
      volumes:
      - name: varlogpods
        hostPath:
          path: /var/log/pods
  
```

Finally we define the command which reads the logs and stream them:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      containers:
      - image: alpine:3.15
        name: alpine
        command:
        - "/bin/sh"
        - "-c"
        - "tail -f /var/log/pods/*/*/*.log"
        volumeMounts:
        - name: varlogpods
          mountPath: /var/log/pods
      volumes:
      - name: varlogpods
        hostPath:
          path: /var/log/pods
```

2. Create the DaemonSet. Where are the pods scheduled ? 

Creation of the DaemonSet

```
k apply -f spec.yaml
```

Listing the pods, we can see no pod is running on the master:

```
k get po -o wide
log-2tzzd   1/1     Running       0          17s   10.38.0.3   worker2   <none>           <none>
log-smtmn   1/1     Running       0          17s   10.32.0.4   worker1   <none>           <none>
```

This is due to the master's taint that the pod does not tolerate. The command below shows the key and effect of that taint:

```
k get no master -o jsonpath={.spec.taints} | jq
[
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/master"
  }
]
```

3. Change the specification so there is a pod on each node of the cluster

We add the toleration so a pod of the DaemonSet can also be scheduled on the master node:

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: log
  name: log
spec:
  selector:
    matchLabels:
      app: log
  template:
    metadata:
      labels:
        app: log
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - image: alpine:3.15
        name: alpine
        command:
        - "/bin/sh"
        - "-c"
        - "tail -f /var/log/pods/*/*/*.log"
        volumeMounts:
        - name: varlogpods
          mountPath: /var/log/pods
      volumes:
      - name: varlogpods
        hostPath:
          path: /var/log/pods
```

Applying the new specification

```
k apply -f spec.yaml
```

There is now one pod per node:

```
k get po -o wide
log-2l2zz   1/1     Running   0          72s   10.40.0.1   master    <none>           <none>
log-676qv   1/1     Running   0          37s   10.32.0.3   worker1   <none>           <none>
log-m8q56   1/1     Running   0          5s    10.38.0.3   worker2   <none>           <none>
```

4. Verify the pod can stream the node's log

Get the log of one of the DaemonSet pod:

```
k logs 2l2zz
```

Or the logs of all DaemonSet pods at once

```
k logs ds/log
```

5. Delete the DaemonSet

```
k delete ds log
```

</details>

