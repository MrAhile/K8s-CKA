Follow the steps below to setup a 3 nodes (1 master / 2 workers) kubeadm cluster .

1. Provisioning the VMs

Note: in this example, we use [Multipass](https://mutlipass.run) a great tool from Canonical to provision local Ubuntu virtual machines, but you can use other tools you are more familiar with.

- creation of the master VM

```
multipass launch -n master -c 2 -m 2G -d 10G
```

- creation of worker1 and worker2 VMs

```
multipass launch -n worker1 -c 2 -m 2G -d 10G
multipass launch -n worker2 -c 2 -m 2G -d 10G
```

Note: by default Multipass create a VM with 1G Ram, 1 CPU and 5G Disk 

For the next steps, I'd recommend you open 4 terminals, one on the host machine and one for each VM.

2. Installation of the dependencies

First, using the following commands, install the dependencies (the container runtime and a couple of libraries) on each node. Also note the version of the cluster is also specified.

- from a shell on the master node:

```
curl https://luc.run/kubeadm/master.sh | VERSION=1.22.7 sh
```

- from a shell on worker1:

```
curl https://luc.run/kubeadm/worker.sh | VERSION=1.22.7 sh
```

- from a shell on worker2

```
curl https://luc.run/kubeadm/worker.sh | VERSION=1.22.7 sh
```

3. Initialisation of the cluster

Init the cluster from a shell on the master node

```
sudo kubeadm init
```

After a few tens of seconds the initialisation will be completed and you will get the command to run on other VMs to add each of them to the cluster. This command will look similar to the following one:

```
kubeadm join 10.96.217.40:6443 --token ucvk2f.vsj6636g36lwwu6x \
	--discovery-token-ca-cert-hash sha256:d772772e54e30d1ae2bc80590d43f3fdf73e1e505b2d551053489786c3338464
```

Still from the master node, retrieve the admin kubeconfig and save it in $HOME/.kube/config (default location to configure kubectl client)

```
mkdir $HOME/.kube
sudo cat /etc/kubernetes/admin.conf > $HOME/.kube/config
```

4. Add worker nodes

Run the above command on worker1 and worker2 to add these nodes to the cluster.

When this is done, go back to the master node and list the cluster's nodes. You should now see the cluster contains 3 nodes:

```
k get no
NAME      STATUS     ROLES                  AGE     VERSION
master    NotReady   control-plane,master   7m7s    v1.22.7
worker1   NotReady   <none>                 4m54s   v1.22.7
worker2   NotReady   <none>                 4m14s   v1.22.7
```

5. Network plugin

The above result shows the cluster is not ready yet, we need to install a network plugin first. In this example we will install WeaveNet but another network plugin could be installed instead.

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=10.32.0.0/16"
```

After a few tens of seconds the cluster will be ready:

```
k get no
NAME      STATUS   ROLES                  AGE     VERSION
master    Ready    control-plane,master   10m     v1.22.7
worker1   Ready    <none>                 8m22s   v1.22.7
worker2   Ready    <none>                 7m42s   v1.22.7
```

6. Get kubeconfig on the host machine

In case you deployed the VM with multipass, use the following command from the host machine to get the admin kubeconfig:

:warning: if on your host machine you already have a *.kube/config* file in your $HOME folder, make sure to backup that one first !

```
multipass exec master -- sudo cat /etc/kubernetes/admin.conf > kubeconfig.cfg
mkdir $HOME/.kube && cp kubeconfig.cfg $HOME/.kube/config
```

You can now communicate with the cluster from the host machine directly (without sshing on the master node anymore)

```
k get no
NAME      STATUS   ROLES                  AGE   VERSION
master    Ready    control-plane,master   13m   v1.22.7
worker1   Ready    <none>                 10m   v1.22.7
worker2   Ready    <none>                 10m   v1.22.7
```
