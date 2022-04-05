## Exercise

1. Create a file private.txt with the content

```
token=cb3456a54EB5
```

2. Create a secret named *credentials*, of type *generic*, from the file private.txt. Make sure the key under the *data* property of the secret is *creds*

3. Create a pod named *test*, with a single container based on the *alpine* image. Make sure this pod has access to the content of the secret in the */secrets/credentials* folder in the container's filesystem

4. Run a shell in the pod's container and verify the content of the */secrets/credentails/creds*

5. Delete the pod and the Secret

<details>
  <summary markdown="span">Solution</summary>

1. Create a file private.txt with the content

```
cat >> private.txt << EOF
token=cb3456a54EB5
EOF
```

2. Create a secret named credentials from this file

```
k create secret generic credentials --from-file=creds=./private.txt
```

The *data* property contains the *creds* key:

```
k get secret credentials -o yaml
apiVersion: v1
data:
  creds: dG9rZW49Y2IzNDU2YTU0RUI1Cg==
kind: Secret
metadata:
  name: credentials
type: Opaque
```

3. Create a pod named *test*, with a single container based on the *alpine* image. Make sure this pod has access to the content of the secret in the */secrets/credentials* folder in the container's filesystem

```
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - image: alpine
    name: alpine
    command:
    - sleep
    - 3600
    volumeMounts:
    - name: creds
      mountPath: /secrets/credentials
  volumes:
  - name: creds
    secret:
      secretName: credentials
```

4. Run a shell in the pod's container and verify the content of the */secrets/credentails/creds*

```
k exec test -- cat /secrets/credentials/creds
token=cb3456a54EB5
```

5. Delete the pod and the Secret

```
k delete po/test secret/credentials
```
</details>

