## Exercise

1. Get the kubelet logs on one of your cluster's worker nodes

2. Check the logs of the control-plane components

3. Where are the control-plane log files located ?

<details>
  <summary markdown="span">Solution</summary>

1. Get the kubelet logs on one of your cluster's worker nodes

From a shell on the node:

```
sudo journalctl -u kubelet
```

2. Check the logs of the control-plane components

If the master node is only named *master*, then you can get the logs using the following commands:

```
# API server
k -n kube-system logs kube-apiserver-master

# Controller Manager
k -n kube-system logs kube-controller-manager-master

# Scheduler
k -n kube-system logs kube-scheduler-master

# etcd
k -n kube-system logs etcd-master
```

3. Where are the control-plane log files located ?

The log files of the control-plane components are located under */var/log/pods/*

```
ls /var/log/pods
kube-system_etcd-master_6d694021cab77267a88779a2268199e6
kube-system_kube-apiserver-master_0a48f4f4304a4deb4d06b553dc09b46c
kube-system_kube-controller-manager-master_94d947d1226129a82876a3b7d829bbfc
kube-system_kube-proxy-25w94_0c17e655-c491-43f6-b012-0eab0c7f8071
kube-system_kube-scheduler-master_415ed7d85341035184628df29257fa2f
kube-system_weave-net-66dtm_cef2efd7-9ea6-4604-a871-53ab915a7a84
...
```

Note: /var/log/pods also contains the log files of any other pods running on the node

</details>




