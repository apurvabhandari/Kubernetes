# Basics of Kubernetes
## CONTROL PLANE COMPONENTS

### KUBE-APISERVER
- Kube-apiserver is Primary Management Component.
- The API Server services REST operations and provides the frontend to the cluster’s shared state through which all other components interact.
- The Kubernetes API server validates and configures data for the api objects which include pods, services, replicationcontrollers, and others.
- It is responsible for Authenticate User, Validate Request, Retrieve data, Update ETCD, Schedular, kubelet.

### ETCD
- ETCD act as cluster database.
- It stores data in the form of Key-Value pair.
- It stores object and config (Nodes, Pods, Configs, Secrets, Accounts, Role, RoleBindings).

### KUBE-CONTROLLER
- Component on the master that runs controllers.
- Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.
- These controllers include:
  - Node Controller
  - Replication Controller
  - Endpoints Controller
  - Service Account & Token Controllers
  
### KUBE-SCHEDULER
- Component on the master that watches newly created pods that have no node assigned, and selects a node for them to run on.
- Factors taken into account for scheduling decisions include individual and collective resource requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference and deadlines.

## NODE COMPONENTS

### CONTAINER RUN TIME INTERFACE (CRI)
- To run containers in Pods, Kubernetes uses a container runtime. 
- At the lowest layers of a Kubernetes node is the software that, among other things, starts and stops containers.
  - Docker
  - CRI-O
  - Containerd
  - RKT
  
### KUBELET
- An agent that runs on each node in the cluster. It makes sure that containers are running in a pod.
- The kubelet takes a set of PodSpecs that are provided through various mechanisms and ensures that the containers described in those PodSpecs are running and healthy. The kubelet doesn’t manage containers which were not created by Kubernetes.

### KUBE-PROXY
- kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.
- kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
- kube-proxy uses the operating system packet filtering layer if there is one and it’s available. Otherwise, kube-proxy forwards the traffic itself.

