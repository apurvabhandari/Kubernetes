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
