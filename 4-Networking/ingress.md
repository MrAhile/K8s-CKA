## Exercise

Let's suppose your cluster is running an NGinx Ingress controller

1. Create the specification of an Ingress resource so that the traffic dedicated to *api.example.com* is redirected to the port 8080 of the internal service named *api*

2. Change the specification so that the filtering is done on *example.com/api* instead of *api.example.com*. Also make sure to remove the */api* before the requested is forwarded to the backend

## Documentation

[https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource)

<details>
  <summary markdown="span">Solution</summary>

1. Create the specification of an Ingress resource so that the traffic dedicated to *api.example.com* is redirected to the port 8080 of the internal service named *api*

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress
spec:
  ingressClassName: nginx
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 8080
```

2. Change the specification so that the filtering is done on *example.com/api* instead of *api.example.com*. Also make sure to remove the */api* before the requested is forwarded to the backend

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ngress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 8080
```

</details>

