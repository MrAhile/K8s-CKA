## Exercise

In this exercice we will show how to upgrade a cluster from kubernetes 1.28.8 to 1.29.3.  
Note: you might need to adapt the content if you have a different version of Kubernetes.

1. Check the version of your cluster

```
$ kubectl get no
NAME            STATUS   ROLES           AGE   VERSION
controlplane    Ready    control-plane   32m   v1.28.8
worker1         Ready    <none>          23m   v1.28.8
worker2         Ready    <none>          18m   v1.28.8
```

Note: in this exercise, we consider a 3 nodes kubeadm cluster (1 controlplane and 2 worker nodes), you'll need to adapt the procedure a little bit if your cluster has a different composition

2. Upgrade the controlplane node

Run a shell on the controlplane node and change the file */etc/apt/sources.list.d/kubernetes.list* so it references version *1.29* instead of *1.28*:

Before:
```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /
```

After:
```
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /
```

Check the latest version of kubeadm

```
sudo apt update
sudo apt-cache madison kubeadm
```

You will be returned the available versions in the *1.29* family. In this exercice we will upgrade to version *1.29.3* but newer versions may exist at the time you are doing this exercice.

First upgrade kubeadm to the new version:

```
VERSION=1.29.3-1.1
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=$VERSION && \
sudo apt-mark hold kubeadm
```

Next simulate the upgrade

```
sudo kubeadm upgrade plan
```

If the previous command went fine we can run the upgrade

```
sudo kubeadm upgrade apply v1.29.3
```

Next drain the controlplane node to prepare it for maintenance

```
kubectl drain controlplane --ignore-daemonsets
```

Next upgrade kubelet and kubectl:

```
VERSION=1.29.3-1.1
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION && \
sudo apt-mark hold kubelet kubectl
```

Then restart kubelet

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Next uncordon the controlplane node

```
kubectl uncordon controlplane
```

If we check the node versions, we can see the controlplane node has been upgraded:

```
$ kubectl get no
NAME            STATUS   ROLES           AGE    VERSION
controlplane    Ready    control-plane   35m    v1.29.3
worker1         Ready    <none>          26m    v1.28.8
worker2         Ready    <none>          21m    v1.28.8
```

Note: if the cluster has several controlplane nodes they need to be upgraded next. The upgrade command would be slightly different and would look like ```kubeadm upgrade NODE_IDENTIFIER```

3. Upgrade the worker nodes

Note: this procedure shows the upgrade of worker1, the same steps must be done for the other worker nodes as well

First run a shell on worker1.

As you did for the controlplane node, change the file */etc/apt/sources.list.d/kubernetes.list* so it references version *1.29* instead of *1.28*.

Next upgrade the kubeadm binary:

```
VERSION=1.29.3-1.1
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=$VERSION && \
sudo apt-mark hold kubeadm
```

Next run the upgrade 

```
sudo kubeadm upgrade node
```

Next, drain the node (run the following command from the controlplane node):

```
kubectl drain worker1 --ignore-daemonsets
```

Then upgrade kubelet and kubectl:

```
VERSION=1.29.3-1.1
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION && \
sudo apt-mark hold kubelet kubectl
```

Then restart kubelet

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Next, uncordon the worker node, making it "schedulable" again (run the following command from the controlplane node):

```
kubectl uncordon worker1
```

If we check the node versions, we can see the controlplane node and the worker1 are upgraded:

```
$ kubectl get no
NAME            STATUS   ROLES           AGE    VERSION
controlplane    Ready    control-plane   36m    v1.29.3
worker1         Ready    <none>          27m    v1.29.3
worker2         Ready    <none>          22m    v1.28.8
```

4. Accessing the upgraded cluster

After the above steps are done on the other worker node, the cluster will be fully upgraded cluster:

```
$ kubectl get no
NAME            STATUS   ROLES           AGE    VERSION
controlplane    Ready    control-plane   38m    v1.29.3
worker1         Ready    <none>          29m    v1.29.3
worker2         Ready    <none>          24m    v1.29.3
```