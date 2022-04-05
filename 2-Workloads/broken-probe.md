## Exercise

1. Create the specification of a Deployment named nginx, with a single replica based on the *nginx:1.20-alpine* image

2. Add a liveness probe which checks every 5 seconds the */* endpoint on port 8080

3. Create the Deployment. What do you notice ?

4. Fix the probe so it checks on port 80

5. Delete the Deployment

<details>
  <summary markdown="span">Solution</summary>

1. Create the specification of a Deployment with a single replica based on the *nginx:1.20-alpine* image

```
kubectl create deploy nginx --image=nginx:1.20-alpine --dry-run=client -o yaml > deploy.yaml
```

2. Add a liveness probe which checks every 5 seconds the */* endpoint on port 8080

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.20-alpine
        name: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          periodSeconds: 5
```

3. Create the Deployment. What do you notice ?

The nginx container is restarted many times.

4. Fix the probe so it checks on port 80

```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx:1.20-alpine
        name: nginx
        livenessProbe:
          httpGet:
            path: /
            port: 80
          periodSeconds: 5
```

5. Delete the Deployment

```
k delete deploy/nginx
```

</details>

