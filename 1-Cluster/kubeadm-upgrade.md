## Exercise

1. Check the version of your cluster

```
kubectl get no
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   32m   v1.24.7
worker1   Ready    <none>                 23m   v1.24.7
worker2   Ready    <none>                 18m   v1.24.7
```

Note: in this exercise, we consider a 3 nodes kubeadm cluster (1 master and 2 worker nodes), you'll need to adapt the procedure a little bit if your cluster has a different composition

2. Upgrade the master node

Run a shell on the master node and check the latest version of kubeadm

```
sudo apt update && sudo apt-cache policy kubeadm
```

The output below specify that we should upgrade the cluster from 1.24.7 to 1.25.3:
```
kubeadm:
  Installed: 1.24.7-00
  Candidate: 1.25.3-00
...
```

First upgrade kubeadm

```
VERSION=1.25.3-00
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=$VERSION && \
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
sudo kubeadm upgrade apply v1.25.3
```

Next we uncordon the master node, making it "schedulable" again

```
kubectl uncordon master
```

Then we upgrade kubelet et kubectl, and restart kubelet:

```
VERSION=1.25.3-00
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION && \
sudo apt-mark hold kubelet kubectl && \
sudo systemctl restart kubelet
```

If we check the node versions, we can see the master node is upgraded:

```
kubectl get no
NAME      STATUS     ROLES           AGE   VERSION
master    NotReady   control-plane   39m   v1.25.3
worker1   Ready      <none>          30m   v1.24.7
worker2   Ready      <none>          30m   v1.24.7
```

Note: if the cluster has several master nodes they need to be upgraded next. The upgrade command would be slightly different and would look like ```kubeadm upgrade NODE_IDENTIFIER```

3. Upgrade the worker nodes

Note: this procedure shows the upgrade of worker1, the same steps must be done for the other worker nodes as well

First run a shell on worker1 and upgrade the kubeadm binary:

```
VERSION=1.25.3-00
sudo apt-mark unhold kubeadm && \
sudo apt-get update && \
sudo apt-get install -y kubeadm=$VERSION && \
sudo apt-mark hold kubeadm
```

Next, drain the node (run the following command from the master node):

```
kubectl drain worker1 --ignore-daemonsets
```

Next, upgrade the kulelet configuration 

```
sudo kubeadm upgrade node
```

Then we upgrade kubelet et kubectl, and restart kubelet:

```
VERSION=1.25.3-00
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && \
sudo apt-get install -y kubelet=$VERSION kubectl=$VERSION && \
sudo apt-mark hold kubelet kubectl && \
sudo systemctl restart kubelet
```

Next, uncordon the worker node, making it "schedulable" again (run the following command from the master node):

```
kubectl uncordon worker1
```

If we check the node versions, we can see the master node and the worker1 are upgraded:

```
kubectl get no
NAME      STATUS   ROLES                  AGE    VERSION
master    Ready    control-plane   42m   v1.25.3
worker1   Ready    <none>          33m   v1.25.3
worker2   Ready    <none>          33m   v1.24.7
```

4. Accessing the upgraded cluster

After the above steps are done on the other worker node, the cluster will be fully upgraded cluster:

```
kubectl get no
NAME      STATUS   ROLES           AGE   VERSION
master    Ready    control-plane   44m   v1.25.3
worker1   Ready    <none>          35m   v1.25.3
worker2   Ready    <none>          35m   v1.25.3
```

