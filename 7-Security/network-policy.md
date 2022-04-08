## Exercise

1. Create a *dev* namespace

2. Create a NetworkPolicy in the *dev* namespace to denies all ingress traffic for every Pod in this namespace

3. Run 2 pods in the *dev* namespace and verify they cannot communicate with each other

4. Create a NetworkPolicy in the *dev* namespace to allow the communication between all Pods in this namespace

5. Delete the dev namespace

## Documentation

[https://kubernetes.io/docs/concepts/services-networking/network-policies/](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

<details>
  <summary markdown="span">Solution</summary>

1. Create a *dev* namespace

```
k create ns dev
```

2. Create a NetworkPolicy in the *dev* namespace to denies all ingress traffic for every Pod in this namespace

Creation of NetworkPolicy:

```
cat <<EOF | k -n dev apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF
```

3. Run 2 pods in the *dev* namespace and verify they cannot communicate with each other

Creation of a nginx pod

```
k -n dev run nginx --image=nginx:1.20
```

Get pod's IP

```
POD_IP=$(k get po nginx -n dev -o jsonpath={.status.podIP})
```

Try to reach the nginx pod from another pod

```
k -n dev run --rm -ti debug --image=alpine:3.15 -- wget ${POD_IP}
...hanging...
```

4. Create a NetworkPolicy in the *dev* namespace to allow the communication between all Pods in this namespace

Creation of a new NetworkPolicy:

```
cat <<EOF | k -n dev apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-in-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector: {}
  egress:
  - to:
    - podSelector: {}
EOF
```

Checking that the debug pod can now reach the nginx one:

```
k -n dev run --rm -ti debug --image=alpine:3.15 -- wget -qO- ${POD_IP}
If you don't see a command prompt, try pressing enter.
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

5. Delete the dev namespace

```
k delete ns/dev
```
</details>

