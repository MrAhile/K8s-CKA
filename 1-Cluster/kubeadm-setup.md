Follow the steps below to setup a 3 nodes (1 controlplane / 2 workers) kubeadm cluster.

1. Pre-requisites

Make sure you have access to 3 virtual machines running Linux and identify these ones as:
- *controlplane*
- *worker1*
- *worker2*

For the next steps, I'd recommend you open 4 terminals, one on the host machine and one for each VM.

2. Installation of the dependencies

First, using the following commands, install the dependencies (the container runtime and a couple of libraries) on each node. Also note the version of the cluster is also specified.

- from a shell on the *controlplane* node:

```
curl https://luc.run/kubeadm/controlplane.sh | VERSION=1.27.5 sh
```

- from a shell on *worker1*:

```
curl https://luc.run/kubeadm/worker.sh | VERSION=1.27.5 sh
```

- from a shell on *worker2*

```
curl https://luc.run/kubeadm/worker.sh | VERSION=1.27.5 sh
```

3. Initialisation of the cluster

Init the cluster from a shell on the *controlplane* node

```
sudo kubeadm init
```

After a few tens of seconds the initialisation will be completed and you will get the command to run on other VMs to add each of them to the cluster. This command will look similar to the following one:

```
kubeadm join 10.96.217.40:6443 --token ucvk2f.vsj6636g36lwwu6x \
	--discovery-token-ca-cert-hash sha256:d772772e54e30d1ae2bc80590d43f3fdf73e1e505b2d551053489786c3338464
```

Still from the *controlplane* node, retrieve the admin kubeconfig and save it in $HOME/.kube/config (default location to configure kubectl client)

```
mkdir $HOME/.kube
sudo cat /etc/kubernetes/admin.conf > $HOME/.kube/config
```

4. Add worker nodes

Run the "sudo kubeadm join..." command (the one you got in the previous step) on *worker1* and *worker2* to add these nodes to the cluster.

Note: if you lost the join command you can generate a new one using: ```sudo kubeadm token create --print-join-command```

When this is done, go back to the *controlplane* node and list the cluster's nodes. You should now see the cluster contains 3 nodes:

```
kubectl get no
NAME            STATUS     ROLES           AGE     VERSION
controlplane    NotReady   control-plane   7m7s    v1.27.5
worker1         NotReady   <none>          4m54s   v1.27.5
worker2         NotReady   <none>          4m14s   v1.27.5
```

5. Network plugin

The above result shows the cluster is not ready yet, we need to install a network plugin first. In this example we will install Cilium but another network plugin could be installed instead.

To install Cilium, run the following commands From the *controlplane* node: 

```
OS="$(uname | tr '[:upper:]' '[:lower:]')"
ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')"
curl -L --remote-name-all https://github.com/cilium/cilium-cli/releases/latest/download/cilium-$OS-$ARCH.tar.gz{,.sha256sum}
sudo tar xzvfC cilium-$OS-$ARCH.tar.gz /usr/local/bin
cilium install
```

After a few tens of seconds the cluster will be ready:

```
$ kubectl get no
NAME            STATUS   ROLES           AGE     VERSION
controlplane    Ready    control-plane   10m     v1.27.5
worker1         Ready    <none>          8m22s   v1.27.5
worker2         Ready    <none>          7m42s   v1.27.5
```