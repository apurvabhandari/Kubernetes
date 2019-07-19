# Install and configure a multi-master Kubernetes cluster with kubeadm 
![img](https://github.com/apurvabhandari/kubernetes/blob/master/kubernets-logo.png)

### Prerequisites
For this lab, we will use a standard Ubuntu 16.04 installation as a base image for the seven machines needed. The machines will all be configured on the same network, 10.10.40.0/24, and this network needs to have access to the Internet.

The first machine needed is the machine on which the HAProxy load balancer will be installed. We will assign the IP 10.10.40.93 to this machine.

We also need three Kubernetes master nodes. These machines will have the IPs 10.10.40.90, 10.10.40.91, and 10.10.40.92.

Finally, we will also have three Kubernetes worker nodes with the IPs 10.10.40.100, 10.10.40.101, and 10.10.40.102.

We also need an IP range for the pods. This range will be 10.30.0.0/16, but it is only internal to Kubernetes.

I will use my Linux desktop as a client machine to generate all the necessary certificates, but also to manage the Kubernetes cluster. If you don't have a Linux desktop, you can use the HAProxy machine to do the same thing.

Installing the client tools
We will need two tools on the client machine: the Cloud Flare SSL tool to generate the different certificates, and the Kubernetes client, kubectl, to manage the Kubernetes cluster.

#### Installing cfssl
1- Download the binaries.

```
$ wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
$ wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
```

2- Add the execution permission to the binaries.

`$ chmod +x cfssl*`

3- Move the binaries to /usr/local/bin.

```
$ sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
$ sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
```

4- Verify the installation.

`$ cfssl version`

### Installing kubectl
1- Download the binary.
```
$wget https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubectl
```
2- Add the execution permission to the binary.
```
$chmod +x kubectl
```
3- Move the binary to /usr/local/bin.
```
$sudo mv kubectl /usr/local/bin
```
4- Verify the installation.
```
$ kubectl version
```

### Installing the HAProxy load balancer
As we will deploy three Kubernetes master nodes, we need to deploy an HAPRoxy load balancer in front of them to distribute the traffic.

1- SSH to the 10.10.40.93 Ubuntu machine.

2- Update the machine.
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
3- Install HAProxy.
```
$ sudo apt-get install haproxy
```
4- Configure HAProxy to load balance the traffic between the three Kubernetes master nodes.
```
$ sudo vim /etc/haproxy/haproxy.cfg
global
...
default
...
frontend kubernetes
bind 10.10.40.93:6443
option tcplog
mode tcp
default_backend kubernetes-master-nodes


backend kubernetes-master-nodes
mode tcp
balance roundrobin
option tcp-check
server k8s-master-0 10.10.40.90:6443 check fall 3 rise 2
server k8s-master-1 10.10.40.91:6443 check fall 3 rise 2
server k8s-master-2 10.10.40.92:6443 check fall 3 rise 2
```
5- Restart HAProxy.
```
$ sudo systemctl restart haproxy
```
