## Exercise

1. List the Pods running the kubernetes dns

2. Show the whole specification of the related Deployment

3. Which ServiceAccount is used by the dns Pods ?

4. Which Roles / ClusterRoles are associated to this ServiceAccount ?

<details>
  <summary markdown="span">Solution</summary>

1. List the Pods running the kubernetes dns

```
k -n kube-system get po -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-78fcd69978-j5cl6   1/1     Running   0          160m
coredns-78fcd69978-pmjl6   1/1     Running   0          160m
```

2. Show the whole specification of the related Deployment

```
k -n kube-system get deploy/coredns -o yaml
```

3. Which ServiceAccount is used by the dns Pods ?

```
k -n kube-system get deploy/coredns -o jsonpath={.spec.template.spec.serviceAccountName}
coredns
```

4. Which Roles / ClusterRoles are associated to this ServiceAccount ?

There is no Roles associated to the coredns ServiceAccount:

```
k get rolebinding -o yaml -A | grep coredns
```

There is a ClusterRole associated to coredns:

```
k get clusterrolebinding system:coredns -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: "2022-04-04T10:41:04Z"
  name: system:coredns
  resourceVersion: "227"
  uid: 4c0c7f71-e3a3-4ab4-8491-c24dfccea4e2
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:coredns
subjects:
- kind: ServiceAccount
  name: coredns
  namespace: kube-system
```

The rules of this ClusterRole are the following ones:

```
k get clusterrole system:coredns -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  creationTimestamp: "2022-04-04T10:41:04Z"
  name: system:coredns
  resourceVersion: "226"
  uid: 3253cce0-51e2-4f8f-b7a2-026bb42b8ab5
rules:
- apiGroups:
  - ""
  resources:
  - endpoints
  - services
  - pods
  - namespaces
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
- apiGroups:
  - discovery.k8s.io
  resources:
  - endpointslices
  verbs:
  - list
  - watch
```

</details>

