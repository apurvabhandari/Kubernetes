# Install and configure a multi-master Kubernetes cluster with kubeadm 
![img](https://github.com/apurvabhandari/kubernetes/blob/master/kubernets-logo.png)

### Prerequisites
For this lab, we will use a standard Ubuntu 16.04 or 18.04 installation as a base image for the seven machines needed. The machines will all be configured on the same network, 10.10.10.0/24, and this network needs to have access to the Internet.

The first machine needed is the machine on which the HAProxy load balancer will be installed. We will assign the IP 10.10.10.93 to this machine.

We also need three Kubernetes master nodes. These machines will have the IPs 10.10.10.90, 10.10.10.91, and 10.10.10.92.

Finally, we will also have three Kubernetes worker nodes with the IPs 10.10.10.100, 10.10.10.101, and 10.10.10.102.

We also need an IP range for the pods. This range will be 10.30.0.0/16, but it is only internal to Kubernetes.

I will use my Linux desktop as a client machine to generate all the necessary certificates, but also to manage the Kubernetes cluster. If you don't have a Linux desktop, you can use the HAProxy machine to do the same thing.

### Installing the client tools
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

#### Installing kubectl
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

#### Installing the HAProxy load balancer
As we will deploy three Kubernetes master nodes, we need to deploy an HAPRoxy load balancer in front of them to distribute the traffic.

1- SSH to the 10.10.10.93 Ubuntu machine.

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
bind 10.10.10.93:6443
option tcplog
mode tcp
default_backend kubernetes-master-nodes


backend kubernetes-master-nodes
mode tcp
balance roundrobin
option tcp-check
server k8s-master-0 10.10.10.90:6443 check fall 3 rise 2
server k8s-master-1 10.10.10.91:6443 check fall 3 rise 2
server k8s-master-2 10.10.10.92:6443 check fall 3 rise 2
```
5- Restart HAProxy.
```
$ sudo systemctl restart haproxy
```
### Generating the TLS certificates
These steps can be done on your Linux desktop if you have one or on the HAProxy machine depending on where you installed the cfssl tool.

#### Creating a certificate authority
1- Create the certificate authority configuration file.
```
$ vim ca-config.json
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
```
2- Create the certificate authority signing request configuration file.
```
$ vim ca-csr.json
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
  {
    "C": "IE",
    "L": "Cork",
    "O": "Kubernetes",
    "OU": "CA",
    "ST": "Cork Co."
  }
 ]
}
```
3- Generate the certificate authority certificate and private key.
```
$ cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```
4- Verify that the ca-key.pem and the ca.pem were generated.
```
$ ls -la
```
#### Creating the certificate for the Etcd cluster
1- Create the certificate signing request configuration file.
```
$ vim kubernetes-csr.json
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
  {
    "C": "IE",
    "L": "Cork",
    "O": "Kubernetes",
    "OU": "Kubernetes",
    "ST": "Cork Co."
  }
 ]
}
```
2- Generate the certificate and private key.
```
$ cfssl gencert \
-ca=ca.pem \
-ca-key=ca-key.pem \
-config=ca-config.json \
-hostname=10.10.10.90,10.10.10.91,10.10.10.92,10.10.10.93,127.0.0.1,kubernetes.default \
-profile=kubernetes kubernetes-csr.json | \
cfssljson -bare kubernetes
```
3- Verify that the kubernetes-key.pem and the kubernetes.pem file were generated.
```
$ ls -la
```
4- Copy the certificate to each nodes.
```
$ scp ca.pem kubernetes.pem kubernetes-key.pem ubuntu@10.10.10.90:~
$ scp ca.pem kubernetes.pem kubernetes-key.pem ubuntu@10.10.10.91:~
$ scp ca.pem kubernetes.pem kubernetes-key.pem ubuntu@10.10.10.92:~
$ scp ca.pem kubernetes.pem kubernetes-key.pem ubuntu@10.10.10.100:~
$ scp ca.pem kubernetes.pem kubernetes-key.pem ubuntu@10.10.10.101:~
$ scp ca.pem kubernetes.pem kubernetes-key.pem ubuntu@10.10.10.102:~
```
### Preparing the nodes for kubeadm

#### Preparing the 10.10.10.90/91/92/100/101/102 machine
Performing below steps on each system
##### Installing Docker latest version
```
$ sudo -s
# curl -fsSL https://get.docker.com -o get-docker.sh
# sh get-docker.sh
# usermod -aG docker your-user
```
#### Installing kubeadm, kublet, and kubectl
1- Add the Google repository key.
```
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
```
2- Add the Google repository.
```
# vim /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io kubernetes-xenial main
```
3- Update the list of packages and install kubelet, kubeadm and kubectl.
```
# apt-get update
# apt-get install kubelet kubeadm kubectl
```
4- Disable the swap.
```
# swapoff -a
# sed -i '/ swap / s/^/#/' /etc/fstab
```
