## Exercise

In this exercice we will show how to upgrade a cluster from kubernetes 1.27.5 to 1.28.2. Please make sure to adapt the content if you have a different version of Kubernetes.

1. Check the version of your cluster

```
$ kubectl get no
NAME            STATUS   ROLES           AGE   VERSION
controlplane    Ready    control-plane   32m   v1.27.5
worker1         Ready    <none>          23m   v1.27.5
worker2         Ready    <none>          18m   v1.27.5
```

Note: in this exercise, we consider a 3 nodes kubeadm cluster (1 controlplane and 2 worker nodes), you'll need to adapt the procedure a little bit if your cluster has a different composition

2. Upgrade the controlplane node

Run a shell on the controlplane node and check the latest version of kubeadm

```
sudo apt update && sudo apt-cache policy kubeadm
```

You will be returned the newly Kubernetes versions. In this exercise we will upgrade the 1.27.5 cluster to 1.28.2.

First upgrade kubeadm to the new version:

```
VERSION=1.28.2-00
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=$VERSION && \
sudo apt-mark hold kubeadm
```

Next drain the controlplane node so pods are removed

```
kubectl drain controlplane --ignore-daemonsets
```

Next simulate the upgrade

```
sudo kubeadm upgrade plan
```

If the previous command went fine we can run the upgrade

```
sudo kubeadm upgrade apply v1.28.2
```

Next uncordon the controlplane node, making it "schedulable" again

```
kubectl uncordon controlplane
```

Then upgrade kubelet and kubectl:

```
VERSION=1.28.2-00
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION && \
sudo apt-mark hold kubelet kubectl && \
sudo systemctl restart kubelet
```

If we check the node versions, we can see the controlplane node has been upgraded:

```
$ kubectl get no
NAME            STATUS   ROLES           AGE    VERSION
controlplane    Ready    control-plane   35m    v1.28.2
worker1         Ready    <none>          26m    v1.27.5
worker2         Ready    <none>          21m    v1.27.5
```

Note: if the cluster has several controlplane nodes they need to be upgraded next. The upgrade command would be slightly different and would look like ```kubeadm upgrade NODE_IDENTIFIER```

3. Upgrade the worker nodes

Note: this procedure shows the upgrade of worker1, the same steps must be done for the other worker nodes as well

First run a shell on worker1 and upgrade the kubeadm binary:

```
VERSION=1.28.2-00
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=$VERSION && \
sudo apt-mark hold kubeadm
```

Next, drain the node (run the following command from the controlplane node):

```
kubectl drain worker1 --ignore-daemonsets
```

Next, upgrade the kulelet configuration 

```
sudo kubeadm upgrade node
```

Then upgrade kubelet and kubectl:

```
VERSION=1.28.2-00
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION && \
sudo apt-mark hold kubelet kubectl && \
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
controlplane    Ready    control-plane   36m    v1.28.2
worker1         Ready    <none>          27m    v1.28.2
worker2         Ready    <none>          22m    v1.27.5
```

4. Accessing the upgraded cluster

After the above steps are done on the other worker node, the cluster will be fully upgraded cluster:

```
$ kubectl get no
NAME            STATUS   ROLES           AGE    VERSION
controlplane    Ready    control-plane   38m    v1.28.2
worker1         Ready    <none>          29m    v1.28.2
worker2         Ready    <none>          24m    v1.28.2
```