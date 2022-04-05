## Purpose

Repository dedicated to the preparation of the CKA certification (Certified Kubernetes Administrator)

## Environment

- Instructor lead training

each participant is provided an Ubuntu virtual machine and will be requested to create a 3 nodes kubeadm cluster inside that one using [https://multipass.run](https://multipass.run) a tool which makes super easy the creation of local Ubuntu VMs.

Note: the host VM has nested virtualization enabled

- self paced training

it is recommended to create 3 local VMs with the tool of your choice (I highly recommend [https://multipass.run](https://multipass.run)) and install a 3 nodes kubeadm cluster (master, worker1 and worker2) on those ones

Cluster setup is detailled in [this exercice](./1-Cluster/kubeadm-setup.md)