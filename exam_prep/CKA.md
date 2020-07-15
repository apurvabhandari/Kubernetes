# Certified Kubernetes Administrator (CKA) with v1.18
<p align="center">
  <img width="300" height="300" src="https://raw.githubusercontent.com/apurvabhandari/kubernetes/master/exam_prep/kubernetes-cka-logo.png">
</p>

- In the Kubernetes certifications (CKAD/CKA) it's important to be fast.<br>
- Before appearing to the exam please go and check out the Latest Kubernetes API version. Exam is based on the latest Kuberntes version only.<br>
- Some of the commands may be deprecated for the latest version and if you try those commands in exam then it will be a big chaos.<br>
- I am writing this blog on the latest version of Kubernetes v1.18 - Kindly ref to [kubernetes.io](https://kubernetes.io) before your exam.<br>
- For CKA you have enough time to solve all 24 questions in 3hrs so no need to alias for even `kubectl=k` or `kubectl get po=kgpo`
- For kubernetes cluster installation of master and worker nodes - Do not go for KTHW (Kubernetes The Hard Way). It is clearly mentioned that install clusters with kubeadm only. So be handy only with kubeadm and kubeadm flags and options.
- Before the exam please just ask the experienced one who appeared for the exam recently.
- I found that bash autocompletion for commands were missing for both root and non-root terminal. Refer [This](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#bash)<br>
- For copy and paste the blogs from docs you do not have a limit for copying and pasting that number of lines. Here to copy paste faster, use keyboard shortcuts copy - `control + insert` for paste - `shift + insert` keep this handy. 
- If you are a heavy linux user please avoid `control + w` this will close your browser session. Only the exam tab and one more tab (kubernetes.io and subdomain) is allowed to open in the browser.
- Before appearing for the exam you should have a basic understanding of Linux and it's compulsory. i.e. `vi/vim`, `cat`, `sudo`, `sudo su -`, `ssh`, `nslookup`, `ping`, `telnet`, `scp`, `mkdir`, `touch`, `awk`, `grep`, `find` or redirection of output into a file, maybe file permission, access denied.
- This is an actual hands-on practical exam. So please do not ask for dumps or questions that appear in the exam.
- For exam instructions please go to cncf official website or you will get at the booking the exam slot.
- Now let's start with actual exam preparation
## Contents:
- Application Lifecycle Management — 8%
- Installation, Configuration & Validation — 12%
- Core Concepts — 19%
- Networking — 11%
- Scheduling — 5%
- Security — 12%
- Cluster Maintenance — 11%
- Logging / Monitoring — 5%
- Storage — 7%
- Troubleshooting — 10%

#### For creating any resource please use dry run at the end and save it in file so that it will be easy to deploy - pod, deployment, service, etc `-o yaml --dry-run=client` <br>
#### In case if you want to change namespace `kubectl config set-context $(kubectl config current-context) --namespace=default`
## Preparation
### 1. Deployment <br>
-   Refer Deployment doc for more details Ref - [kubernetes deployment](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)<br>
-   To create simple deployment <br>
   `kubectl create deployment nginx-deploy --image=nginx` <br>
-   Custom deployment if you want to edit or modify. Open deployment.yaml make the changes and save it.<br>
   `kubectl create deployment nginx-deploy --image=nginx -o yaml --dry-run=client > deployment.yaml` <br>
-   Once changes saved then,<br>
   `kubectl apply -f deployment.yaml`<br>
-   To verify deployment,<br>
   `kubectl get deployment`<br>
   `kubectl describe deployment nginx-deploy`<br>
-   To scale deployment = 2<br>
   `kubectl scale deployment nginx-deploy --replicas=2`<br>
-   To expose deployment on port = 80 as a ClusterIP or accessing service within the Cluster<br>
   `kubectl expose deployment nginx-deploy --name=nginx-service --port=80 --target-port=80 --type=ClusterIP`<br>
   `kubectl get svc`<br>
-   Here is the sample deployment using dry run<br>
      `kubectl create deployment nginx --image=nginx:1.12 -o yaml --dry-run=client`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  selector:
    matchLabels:
      app: nginx-deply
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - name: nginx
        image: nginx:1.12
        ports:
        - containerPort: 80
```
   `kubectl apply -f deployment.yaml`
-   Deployment rollout and undo rollout<br>
   a. Deploying nginx:1.12 <br>
      `kubectl create deployment nginx-deploy --image=nginx:1.12`<br>
   b. Updating image to 1.13 i.e. rollout<br>
       Check the name of nginx container<br>
      `kubectl describe deployment nginx-deploy`<br>
      `kubectl set image deployment/nginx-deploy nginx=nginx:1.13 --record=true`<br>
   c. Check rollout history<br>
      `kubectl rollout history deployment nginx-deploy`<br>
   d. Undo rollout<br>
      `kubectl rollout undo deployment nginx-deploy`<br>
      
   ##### Note: Do not use `kubectl run nginx --image=nginx --restart=Always` to create deployment as it's been deprecated in v1.18<br>
   
### 2. Pod <br>
- Create pod Ref - [kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pod-templates)<br>
  `kubectl run nginx-pod --image=nginx`<br>
- Edit or modify before runing pod by dry-run<br>
  `kubectl run nginx-pod --image=nginx -o yaml --dry-run=client > pod.yaml`<br>
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
```
- Expose pod - please check deployment expose<br>
- Check exposed pod from another pod or accessing one pod from another pod<br>
`kubectl run tmp --image=busybox:1.28.0 --restart=Never --rm -it ping nginx-service`<br>
`kubectl run tmp --image=busybox:1.28.0 --restart=Never --rm -it nslookup nginx-service`<br>
check with pod - check the pod ip consider here it is 10.0.0.1 and namespace = default<br>
`kubectl run tmp --image=busybox:1.28.0 --restart=Never --rm -it nslookup 10-0-0-1.default.pod`<br>
- StaticPod Ref- [kubernetes StaticPod](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)<br>
  a. create pod by dry-run<br>
     `kubectl run nginx-pod --image=nginx -o yaml --dry-run=client > staticpod.yaml`<br>
  b. copy staticpod.yaml file by `scp` to worker node and keep in `/etc/kubernetes/manifest` or check staticPodPath path from config.yaml and place yaml file on the same<br>
  c. check pod is up or not<br>
     `kubectl get pod`<br>
- Multi-container pod with nginx and redis - copy spec from pod template<br>
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: multicontainer
  name: multicontainer
spec:
  containers:
  - image: nginx
    name: nginx
  - image: redis
    name: redis
```
- Troubleshoot over pod or deployment<br>
   a. Check pod logs <br>
      `kubectl logs nginx`<br>
   b. Check pod failure <br>
      `kubectl describe pod nginx`<br>
   c. Edit pod<br>
      `kubectl edit pod nginx`<br>
   d. login into pod<br>
      `kubectl exec -it nginx sh`<br>
   
### 3. initContainer<br>
- Create a normal pod by dry-run and add initContainer spec in it. Here init container is downloading index.html before start of actual nginx container Ref - [kubernetes initContainer](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/#create-a-pod-that-has-an-init-container)<br>
```
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: workdir
      mountPath: /usr/share/nginx/html
  # These containers are run during pod initialization
  initContainers:
  - name: install
    image: busybox
    command:
    - wget
    - "-O"
    - "/work-dir/index.html"
    - http://kubernetes.io
    volumeMounts:
    - name: workdir
      mountPath: "/work-dir"
  dnsPolicy: Default
  volumes:
  - name: workdir
    emptyDir: {}
```
### 4. DaemonSet <br>
- Create DaemonSet or Deploy DaemonSet so that every node contain at least have 1 pod running on it Ref - [kubernetes DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec)<br>
```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
```
### 5. Secret<br>
- Create secret name=mysecret from username:testuser, password:iluvtests Ref - [kubernetes secret](https://kubernetes.io/docs/concepts/configuration/secret/)<br>
  `kubectl create secret generic mysecret --from-literal=username=testuser --from-literal=password=iluvtests`
- secret as a environment variables SECRET_USERNAME
```
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
```
- secret as a volume mount path /etc/foo 
```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

### 6. PersistentVolume & PersistentVolumeClaim <br>
- Create pv Ref - [kubernetes pv](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)<br>
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
`kubectl get pv`
- Create pvc - it should be in a bound state once pvc is created. Remember storageClassName should be same as pv and also check for accessModes
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
`kubectl get pvc`
- Use above pvc in pod
```
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```
- pod with emptyDir{} i.e. data should be deleted if pod delete and mount on /cache Ref - [kubernetes emptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#example-pod)<br>
```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

### 7. Network policy <br>
- Here we need to understand the scenario and create a network policy or label pod. Ref - [kubernetes network policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/)<br>
`kubectl get pods --show-labels`<br>
`kubectl label pod redis role=db`
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
```

### 8. SecurityContext<br>
- Set capabilities for Container Ref - [kubernetes security context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)<br>
```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo-4
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```
- securityContext runAsUser: 1000 for pod
```
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
```

### 9. RBAC<br>
- csr and approve csr from .csr file Ref - [kubernetes csr](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)<br>
```
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: my-svc
spec:
  request: $(cat server.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
```
`kubectl certificate approve my-svc`
- create role who can get, list, watch, update, delete the pod Ref - [kubernetes rbac](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities)<br>
`kubectl create role devuser --verb=get,list,watch,update,delete --resource-name=pods`<br>
- create rolebindings<br>
`kubectl create rolebinding newrole --clusterrole=view --serviceaccount=default:myapp --namespace=default`<br>

### 10. ETCD Backup<br>
- Taking backup Ref - [kubernetes etcd backup](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#built-in-snapshot)<br>
```
$export ETCDCTL_API=3
## check options with help command
$etcdctl --help
## copy the cert path from below command
$kubectl describe pod -n kube-system etcd-master
## please verify the path from above command 
$etcdctl snapshot save snapshotdb.db --endpoints=127.0.0.1:2379 --cacert="/etc/kubernetes/pki/etcd/ca.crt" --cert="/etc/kubernetes/pki/etcd/server.crt" --key="/etc/kubernetes/pki/etcd/server.key"
```
- Verify snapshot<br>
  `etcdctl --write-out=table snapshot status snapshotdb.db`<br>

### 11. Cluster Installation<br>
- kubeadm installation on both node Master and Worker Node. `ssh` to master and worker node for installation. Ref - [kubernetes kubeadm installation](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)<br>
```
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
systemctl daemon-reload
systemctl restart kubelet
```
- kubeadm init to initialize cluster check if any arguments has asked while initialize the cluster - Master node only Ref - [kubernetes kubeadm initialize](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)<br>
```
kubeadm init <args>
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
- Join Worker node - Copy this from init command and run this on Worker node Only<br>
`  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>`<br>
- Confirm nodes it should be in Ready state <br>
`kubectl get nodes`<br>

### 12. Cluster troubleshoot<br>
- Check master node and worker is running or not<br>
- Check all components are running or not. like ETCD, kubelet, kube scheduler, etc<br>
- Check all paths verifying all are on the correct path in config and actual. like cert path, config path, kubeconfig file<br>
- Check logs of master components which are failing.<br>
- If the component is running on system like kubelet then check in `/var/log/messages` or `/var/log/syslog` or `journalctl -u kubelet` <br>

### 13. jsonpath<br>
- Examples Ref - [kubernetes jsonpath](https://kubernetes.io/docs/reference/kubectl/jsonpath/)<br>
```
# Get the version label of all pods with label app=cassandra
kubectl get pods --selector=app=cassandra -o  jsonpath='{.items[*].metadata.labels.version}'
# Get ExternalIPs of all nodes
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'
# List Events sorted by timestamp
kubectl get events --sort-by=.metadata.creationTimestamp
# All images running in a cluster
kubectl get pods -A -o=custom-columns='DATA:spec.containers[*].image'
# sort by capacity
kubectl get pv --sort-by=.spec.capacity.storage
```
### 14. [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)<br>

## Common Mistakes
- Make sure that you have switched to correct context/cluster or not. That is mentioned on top of every question.
- Wrong indentation in yaml file.
- Deployment in wrong namespace.
- Checking pod in wrong namespace.
- Adding labels to deployment and pod.
- Recheck or verify the deployment or pod by 1/1 or 2/2 in Ready state.
- If you are applying any yaml file then check if all mandatory fields are present or not. Eg DaemonSet or Network Policy.
- Check deployment for DaemonSet, whether it is running on all nodes or not.
- For initContainer the container is up or not with 1/1 after Init1:1
- For jsonpath output, check whether the output is in correct format or not as per question.
- While storing output checks you have access to the mentioned location and once stored verify with `cat` command.
- Do not waste your time solving any question where you stuck for a long time.
- Instead of creating manual yaml files, use kubernetes.io and copy paste the block in the file editor.
- For troubleshooting/installation of clusters, check nodes are up and running.
- Most Important - Do not delete or recreate existing resources. You are only allowed to modify it. Do not try to export, delete and create those resources. Where you can create, delete, update your resources that have been asked in exam. 

### All the Best For your Exam!!

## My CKA certificate

<p align="center">
  <img src="https://raw.githubusercontent.com/apurvabhandari/kubernetes/master/exam_prep/CKA_Apurva.png">
</p>

### Creator
Do reach out to me for any query regarding the exam.
- [Apurva Bhandari](https://www.linkedin.com/in/apurvabhandari-linux/)<br>

How to start ? <br>
- Check Learning Resources [References](https://github.com/apurvabhandari/Kubernetes/blob/master/exam_prep/README.md#references)<br>
