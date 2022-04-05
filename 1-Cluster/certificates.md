## Exercise

1. On a kubeadm cluster, where are located the private keys and related certificates used by the control-plane components to communicate with each other ?

2. Where is the certificate used by kubelet to communicate with the API Server ?

3. Using openssl, get the Group and User that identifiy the kubelet agent

4. Get the rights associated to the previous Group

<details>
  <summary markdown="span">Solution</summary>

1. On a kubeadm cluster, where are located the private keys and related certificates used by the control-plane components to communicate with each other ?

The PKI are located on the master nodes in the folder */etc/kubernetes*

```
find /etc/kubernetes
/etc/kubernetes
/etc/kubernetes/kubelet.conf
/etc/kubernetes/controller-manager.conf
/etc/kubernetes/manifests
/etc/kubernetes/manifests/kube-controller-manager.yaml
/etc/kubernetes/manifests/kube-scheduler.yaml
/etc/kubernetes/manifests/etcd.yaml
/etc/kubernetes/manifests/kube-apiserver.yaml
/etc/kubernetes/scheduler.conf
/etc/kubernetes/pki
/etc/kubernetes/pki/front-proxy-ca.key
/etc/kubernetes/pki/front-proxy-client.key
/etc/kubernetes/pki/ca.key
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/front-proxy-client.crt
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/apiserver-kubelet-client.key
/etc/kubernetes/pki/sa.key
/etc/kubernetes/pki/apiserver-etcd-client.key
/etc/kubernetes/pki/apiserver.key
/etc/kubernetes/pki/etcd
/etc/kubernetes/pki/etcd/ca.key
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/peer.key
/etc/kubernetes/pki/etcd/healthcheck-client.key
/etc/kubernetes/pki/etcd/healthcheck-client.crt
/etc/kubernetes/pki/etcd/server.key
/etc/kubernetes/pki/etcd/peer.crt
/etc/kubernetes/pki/apiserver-etcd-client.crt
/etc/kubernetes/pki/sa.pub
/etc/kubernetes/pki/front-proxy-ca.crt
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/admin.conf
```

Some certificates / keys are under the *pki* subfolder, other ones are defined in the kubeconfig files (.conf extension)

2. Where is the certificate used by kubelet to communicate with the API Server ?

The certificate is defined in the */etc/kubernetes/kubelet.conf* file

```
sudo cat /etc/kubernetes/kubelet.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMvakNDQWVhZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeU1ETXlOVEUyTURVeU0xb1hEVE15TURNeU1qRTJNRFV5TTFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBT1d0CjYySjErK1pONGRXYm1MZlJIUkZ0aVhoVW5YQ1BRbkdWMFVVbFlTSlFwS1VRSndiVDM0UjQ4bUZ1eVpkWEEzNUEKSGtlVkpSM214RndYVjdQWmtqV2RHRWNsT2hWdFp6UVR4akM0eEwyRkdEL2srNFhJbUdobWJycWRMd3Yzdm1NNwpONFUvcjIrMUQ5MDExdkkvVVlZQXc4Q2xwUll2KzErZkZrZDR1YlRIY3VnMW9sRUR1bTVzekZQaUx1RlRYREJYCjYvWEJhZzdKWU1Wc3d2MHpGa3kxTzJFRFI0NDRCVG9hbGN1djVucGdpdWFnTlptV3laYnNKWURyYlBrV2t6Nk8KZE5ZcmFXTXNSckJRcFgvVjl0NkRuUzc2eVdQS3ZhUUZzbkNLVnFMRzVPWm44c0lKMUprN2JGV2wzVWNsYlIrQwpCdXFwSWZZNFZJVS90ZWpsOTUwQ0F3RUFBYU5aTUZjd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZLZm9uajAvWURhVjBCZXo2djVXT2xydG1ReWhNQlVHQTFVZEVRUU8KTUF5Q0NtdDFZbVZ5Ym1WMFpYTXdEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBTEtQRlNwNHN4b3ZxTllVY25YVApnbEZ4NkdMMC9STXptM251cTVSSitYdGtqUkZJczhXVERhS0dRaXdRbU56Mm9obFExcHM3MmdJaE5nbzcrYXZtCktudGZCVGpQR2R3YXE3bStGd0c3TGh1Y1pQL0hLb3BqN2dqbllJbFBjcjhiSXRlUi9nUE12NDJQSE93VzVnQkQKdkd2SFJpMXJ6dEdLcnZlS1FPOEpycU44Wm83bzMxc2RrZ20xcTdsNUFERTdxTEJiUzVXQmxtd2hWeE5NQUI4cApqZnpMWFFwK0FWazVzQnJWZzF2c0UzbXR3SzVkR29sQkNaMUxPVGtzVGcrdDA1eW0zOUlFdkRSdnJ2RS9EVzJFCitOY3FOY254RGtHdUVMc2pmUGgzeE9pWUpKelFBNzVhaGFERnAyWFJPUDZjU1FZU3VIM2x1Wk1pOXFBclhPSjUKenhBPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://194.182.171.68:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: system:node:master
  name: system:node:master@kubernetes
current-context: system:node:master@kubernetes
kind: Config
preferences: {}
users:
- name: system:node:master
  user:
    client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
    client-key: /var/lib/kubelet/pki/kubelet-client-current.pem
```

In this example, the certificate is located at */var/lib/kubelet/pki/kubelet-client-current.pem*

3. Using openssl, get the Group and User that identify the kubelet agent

Using openssl to get the content of the certificate:

```
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4838438186102467812 (0x432595e52652ace4)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kubernetes
        Validity
            Not Before: Mar 25 16:05:23 2022 GMT
            Not After : Mar 25 16:05:26 2023 GMT
        Subject: O = system:nodes, CN = system:node:master
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:95:70:d8:6e:1a:5c:6e:c9:1f:b5:00:a2:ed:d6:
                    03:f2:9f:f1:6a:44:4c:18:7e:ef:69:3e:d1:67:39:
                    1c:70:cb:68:5e:e0:5e:4c:ab:d0:82:8f:a9:30:4c:
                    80:e9:52:83:7c:26:4d:70:27:9e:67:51:0a:0b:cf:
                    c4:a7:ac:3c:be:4c:2e:15:8d:27:24:78:0c:6d:59:
                    f3:cb:4b:6a:bd:b7:b1:98:b7:d6:71:b6:a3:ff:a5:
                    0d:56:f8:a5:03:8d:ef:d3:4d:68:a1:60:45:8d:2f:
                    df:c0:c0:b8:b5:3c:e1:db:74:66:f6:69:c7:2b:9c:
                    cd:dc:3f:a5:84:1e:60:71:65:24:9b:33:5c:29:33:
                    9e:fe:c0:01:40:f5:bc:ce:fd:e0:ee:f6:7e:4b:32:
                    51:3f:69:01:5b:cd:cf:36:eb:2c:89:10:ae:a8:75:
                    46:96:ef:ba:d8:aa:1f:b0:1a:ec:a5:91:bb:88:8f:
                    4a:67:55:0e:57:8e:27:66:ae:da:7d:41:18:27:04:
                    99:1c:c6:64:4c:c1:9a:36:89:e6:88:3c:76:ad:6e:
                    cd:dc:19:3d:bf:f7:51:86:92:a6:e1:d4:9f:55:32:
                    28:3d:ed:0f:65:07:71:4b:96:36:87:a3:b1:57:ad:
                    86:27:4f:22:20:6c:8d:c1:9e:13:4a:55:d3:e2:81:
                    14:59
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                keyid:A7:E8:9E:3D:3F:60:36:95:D0:17:B3:EA:FE:56:3A:5A:ED:99:0C:A1

    Signature Algorithm: sha256WithRSAEncryption
         09:b1:f0:66:0d:1b:0e:0f:29:3f:84:47:d2:a4:7c:86:99:a9:
         83:a5:8c:f8:98:75:c9:d3:4b:27:b5:01:bf:15:d5:df:1e:ed:
         a8:3d:55:d4:e5:d2:f8:d2:e0:45:6e:ac:d1:b4:cb:6a:d9:d5:
         22:df:2f:0d:67:9b:9f:d3:1a:39:01:9d:30:c0:90:f1:44:d8:
         61:6e:f5:ae:3a:d5:66:09:c6:3e:d1:d9:82:37:99:d5:9a:4b:
         f6:70:e9:32:eb:12:ab:16:4b:84:9c:e2:32:f0:70:3d:76:2e:
         d7:13:f6:fb:05:53:ab:60:7c:e4:b3:14:ba:6d:5d:b5:33:33:
         34:d3:bf:16:40:3e:e2:83:a7:2d:99:7e:55:de:6e:43:40:7c:
         7b:62:95:e2:da:1c:9d:ad:b5:62:14:21:5b:2e:49:17:b4:a0:
         e4:1d:3d:c9:9a:d1:c2:f7:38:3c:9b:c0:a9:1d:d9:a1:e3:4a:
         dd:fc:37:3b:1c:3a:d0:43:de:5e:f1:c7:f7:48:c4:d3:0c:d8:
         46:73:2a:b8:57:50:3f:bc:dd:53:de:d9:5a:9f:fa:65:cb:b5:
         ed:c4:9a:e1:68:19:65:73:c1:04:57:fc:cf:bd:9d:da:e4:68:
         d1:f2:23:90:a8:56:ed:b7:de:41:84:bf:af:7b:d2:21:61:52:
         11:e9:03:67
```

From the subject information, we can get the Group and User which authenticates the kubelet agent:

```
=> Group is *system:nodes*
   User is *system:node:master*
```

4. Get the rights associated to the previous Group

Getting the ClusterRole with the same name as the Group (*system:node*):

```
k get clusterrole system:node  -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2022-03-25T16:05:49Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:node
  resourceVersion: "95"
  uid: 430c684e-54a0-4eeb-96ae-e29fc9c2a7a5
rules:
- apiGroups:
  - authentication.k8s.io
  resources:
  - tokenreviews
  verbs:
  - create
- apiGroups:
  - authorization.k8s.io
  resources:
  - localsubjectaccessreviews
  - subjectaccessreviews
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - services
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - create
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - events
  verbs:
  - create
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - create
  - delete
- apiGroups:
  - ""
  resources:
  - pods/status
  verbs:
  - patch
  - update
- apiGroups:
  - ""
  resources:
  - pods/eviction
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - configmaps
  - secrets
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - persistentvolumeclaims
  - persistentvolumes
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - endpoints
  verbs:
  - get
- apiGroups:
  - certificates.k8s.io
  resources:
  - certificatesigningrequests
  verbs:
  - create
  - get
  - list
  - watch
- apiGroups:
  - coordination.k8s.io
  resources:
  - leases
  verbs:
  - create
  - delete
  - get
  - patch
  - update
- apiGroups:
  - storage.k8s.io
  resources:
  - volumeattachments
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - serviceaccounts/token
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - persistentvolumeclaims/status
  verbs:
  - get
  - patch
  - update
- apiGroups:
  - storage.k8s.io
  resources:
  - csidrivers
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - storage.k8s.io
  resources:
  - csinodes
  verbs:
  - create
  - delete
  - get
  - patch
  - update
- apiGroups:
  - node.k8s.io
  resources:
  - runtimeclasses
  verbs:
  - get
  - list
  - watch
  ```

As we can see, the kubelet agent is allowed to perform many actions on the cluster
</details>

