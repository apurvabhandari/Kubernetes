
Kube-proxy can work in three modes:
- userspace

In this mode, Kube-proxy installs iptables rules which capture traffic to a Service’s ClusterIP and redirects that traffic to Kube-proxy’s listening port. Kube-proxy then chooses a backend Pod and forwards the request to it.
kube-proxy serves as an OSI layer 4 load balancer in this model. Since Kube-proxy runs in the userspace, packages need to be copied back and forth between kernelspace and userspace, adding extra latency in the proxy process. The advantage is that Kube-proxy can retry other Pod if the first Pod is not available.

- iptables

To avoid the additional copies between kernelspace and userspace, Kube-proxy can work on iptables mode. Kube-proxy creates an iptables rule for each of the backend Pods in the Service. After catching the traffic sent to the ClusterIP, iptables forwards that traffic directly to one of the backend Pod using DNAT. In this mode, Kube-proxy no longer serves as the OSI layer 4 proxy. It only creates corresponding iptables rules. Without switching between kernelspace and userspace, the proxy process is more efficient.

- ipvs

This model is similar to iptables because both ipvs and iptables are base on netfilter hook in kernelspace. Ipvs uses hash tables to store rules, meaning it’s faster than iptables, especially in a large cluster where there’re thousands of services. In addition, ipvs supports more load balancing algorithms.
