---
layout: post
title:  "Bare Metal Kubernetes Cluster"
date:   2020-05-06 00:50:05 +0530
categories: kubrnetes
---

Kubernetes is an open source project initiated by google to ease the pain of deploying, managing and scaling containerized applications. There exists many flavors of kubernetes clouds such as

- GKE (Google’s very own kubernetes engine)
- AWS Eks
- Openshift

If you are new to kubernetes and feel the need to experiment you could always try the single node solutions given below or utilize the free credit on GKE.

- Minikube
- Minishift

However if you are feeling a bit more adventurous and have some computing power to spare you could always bootstrap your own kubernetes cluster effortlessly. You would need two machines at-least for this and static IPs within the network to find them.

In this single node setup one machine will be the master while the other a node. All nodes(including the master) requires some basic tools to create and join the cluster. Docker, kubelet, kubectl(optional). In addition to those the master will need kubeadm to administrate the cluster.

If you are planning to access the cluster from a different machine (most probable scenario) you would need kubectl installed in that machine. kubectl is a kubernetes client application written in GO that could talk to the kubernetes API that we are going to setup in the master node.

# Setting up the master

SSH into the master machine and use the following commands to install the required tools.

```bash
apt-get update && apt-get install -y apt-transport-https
curl -s https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository \
"deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
apt update && apt install -qy docker-ce
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" \
    > /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubeadm kubelet kubectl
```

Use **KUBEADM** to initialise the cluster using the following command where **<MASTER_IP>** is the IP of the master 
machine that the nodes could find and –pod-network-cidr could be any range of IP addresses that the pods inside the clusters can be assigned to. This network range is isolated from the external network and only used for inter-cluster communication. However it should be noted that assigning the IP range as same as your network could cause problems.

```bash
kubeadm init --apiserver-advertise-address=<MASTER_IP> --pod-network-cidr=192.168.1.0/16
```

After the command has run, you will find a command in the output that can be used by the worker nodes to join the cluster initialized by the master.

In addition, you need to setup a CIDR for maintaining the internal network. You could find a few options if you skim through the kubernetes documentation. We chose to go with flannel which could be installed using the command given below.

# Setting up the worker node(s)

This is fairly straight forward. Install the dependencies as before using the following commands and use the join command we got earlier.

```bash
 kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
This is fairly straight forward. Install the dependencies as before using the following commands and use the join command we got earlier.

```bash
apt-get update && apt-get install -y apt-transport-https
curl -s https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
apt update && apt install -qy docker-ce
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" \
    > /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y kubeadm kubelet kubectl
```

If you’ve missed it or you want to join a new node to the cluster, use the following command to get a new token along with the join command.

```bash
sudo kubeadm token create --print-join-command
```

# Give it a go!

With the kubernetes configuration we copied earlier from the master, we are now able to connect to the cluster and administrate it using the kubernetes API. To test the nodes in the cluster, use the following command.

```bash
kubectl get nodes
```

If all went well, you should see two nodes including the master and a worker node. You could always expand your cluster by adding new nodes to it by following the steps in the above section.