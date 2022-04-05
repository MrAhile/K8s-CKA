## Exercise

1. Install metrics server with the *--kubelet-insecure-tls* flag and wait for it to work fine.

Note: it can be installed from [https://luc.run/metrics-server.yaml](https://luc.run/metrics-server.yaml)

2. Create the Deployment *www* with a single replica based on nginx and expose the Pods with a ClusterIP Service named *www*

3. Create a HorizontalPodAutoscaller with a target CPU of 50% and with a minimum of 3 and a maximum of 10 replicas

4. Run a stress Pod (command below) and verify the number of replicas are increased

```
k run ab -ti --rm --restart='Never' --image=lucj/ab -- -n 100000 -c 50 http://www/
```

5. Delete the HPA, the Deployment and the Service

<details>
  <summary markdown="span">Solution</summary>

1. Install metrics server with the *--kubelet-insecure-tls* flag and wait for it to work fine.

Installation:

```
k apply -f https://luc.run/metrics-server.yaml
```

Metrics is running fine when it returns nodes / pods metrics:

```
k top node
NAME      CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master    119m         5%     1242Mi          65%
worker1   33m          1%     962Mi           51%
worker2   48m          2%     987Mi           52%
```

```
k top pod
NAME    CPU(cores)   MEMORY(bytes)
mongo   5m           68Mi
nginx   0m           3Mi
```

2. Create a Deployment with a single replica based on nginx and expose the Pods with a Service named *www*

Creation of the Deployment

```
k create deploy www --image=nginx:1.20
```

Exposition with a Service

```
k expose deploy/www --name=www --port=80
```

3. Create a HorizontalPodAutoscaller with a target CPU of 50% and with a minimum of 3 and a maximum of 10 replicas

```
k autoscale deploy www --min=3 --max=10 --cpu-percent=50
```

4. Run a stress Pod (command below) and verify the number of replicas are increased

```
k run ab -ti --rm --restart='Never' --image=lucj/ab -- -n 100000 -c 100 http://www/
```

Get the number of replicas:

```
kubectl get pod
```

5. Delete the HPA, the Deployment and the Service

```
k delete deploy/www svc/www hpa/www
```

</details>

