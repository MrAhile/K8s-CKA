## Exercise

1. Check the version of your cluster

```
k get no
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   32m   v1.22.7
worker1   Ready    <none>                 23m   v1.22.7
worker2   Ready    <none>                 18m   v1.22.7
```

Note: in this exercise, we consider a 3 nodes kubeadm cluster (1 master and 2 worker nodes), you'll need to adapt the procedure a little bit if your cluster has a different composition

2. Upgrade the master node

Check the latest version of kubeadm

```
sudo apt update && sudo apt-cache policy kubeadm
```

The output below specify that we should upgrade the cluster from 1.22.7 to 1.23.5:
```
kubeadm:
  Installed: 1.22.7-00
  Candidate: 1.23.5-00
...
```

First upgrade kubeadm

```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=1.23.5-00 && \
sudo apt-mark hold kubeadm
```

Next drain the master node so pods are removed

```
kubectl drain master --ignore-daemonsets
```

Next we simulate the upgrade

```
sudo kubeadm upgrade plan
```

If the previous command went fine we can run the upgrade

```
sudo kubeadm upgrade apply v1.23.5
```

Next we uncordon the master node, making it "schedulabel" again

```
kubectl uncordon master
```

Then we upgrade kubelet et kubectl, and restart kubelet:

```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=1.23.5-00 kubectl=1.23.5-00 && \
sudo apt-mark hold kubelet kubectl && \
sudo systemctl restart kubelet
```

If we check the node versions, we can see the master node is upgraded:

```
k get no
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   119m   v1.23.5
worker1   Ready    <none>                 110m   v1.22.7
worker2   Ready    <none>                 105m   v1.22.7
```

Note: if the cluster has several master nodes they need to be upgraded next. The upgrade command would be slightly different and would look like ```kubeadm upgrade NODE_IDENTIFIER```

3. Upgrade the worker nodes

Note: this procedure shows the upgrade of worker1, the same steps must be done for the other worker nodes as well

First upgrade the kubeadm binary:

```
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=1.23.5-00 && \
sudo apt-mark hold kubeadm
```

Next, drain the node (this command cannot be run from the worker)

```
kubectl drain worker1 --ignore-daemonsets
```

Next, upgrade the kulelet configuration 

```
sudo kubeadm upgrade node
```

Then we upgrade kubelet et kubectl, and restart kubelet:

```
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=1.23.5-00 kubectl=1.23.5-00 && \
sudo apt-mark hold kubelet kubectl && \
sudo systemctl restart kubelet
```

Next, uncordon the worker node, making it "schedulabel" again (this command cannot be run from the worker):

```
kubectl uncordon worker1
```

If we check the node versions, we can see the master node and the worker1 are upgraded:

```
k get no
NAME      STATUS   ROLES                  AGE    VERSION
master    Ready    control-plane,master   135m   v1.23.5
worker1   Ready    <none>                 125m   v1.23.5
worker2   Ready    <none>                 120m   v1.22.7
```

4. Accessing the upgraded cluster

After the above steps are done on the other worker node, the cluster will be fully upgraded cluster:

```
k get no
NAME      STATUS   ROLES                  AGE    VERSION
master    Ready    control-plane,master   138m   v1.23.5
worker1   Ready    <none>                 129m   v1.23.5
worker2   Ready    <none>                 124m   v1.23.5
```

