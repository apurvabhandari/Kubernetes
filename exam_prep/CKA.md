# Certified Kubernetes Administrator (CKA)
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
- For copy and paste the blogs from docs you do not have a limit for copying and pasting that number of lines. Here to copy paste faster, use keyboard shortcuts copy - `control+ insert` for paste - `shift + insert` keep this handy.
- Before appearing for the exam you should have a basic understanding of Linux and it's compulsory. i.e. `vi/vim` a file editor, `cat`, `sudo`, `sudo su -`, `ssh`, `nslookup`, `ping`, `telnet`, `scp`, `mkdir`, `touch`, `awk`, `grep`, `find` or redirection of output into a file, maybe file permission, access denied.
- This is an actual hands-on practical exam. So please do not ask for dumps or questions that appear in the exam.
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
-   Refer Deployment doc for more details - [kubernetes deployment](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/)<br>
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
      
   ##### Note: Do not use `kubectl run nginx --image-nginx --restart=Always` to create deployment as it's been deprecated in v1.18<br>
   
### 2. Pod <br>
- Create pod Ref [kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pod-templates)<br>
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
- StaticPod Ref [kubernetes StaticPod](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)<br>
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
- Troubleshoot over pod or deployment
   a. Check pod logs <br>
      `kubectl logs nginx`<br>
   b. Check pod failure <br>
      `kubectl describe pod nginx`<br>
   c. Edit pod
      `kubectl edit pod nginx`
   d. login into pod
      `kubectl exec -it nginx sh`
   
### 3. initContainer<br>
- Create normal pod by dry-run and add initContainer spec in it. Here init container is downloading index.html before start of actual nginx container Ref - [kubernetes initContainer](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/#create-a-pod-that-has-an-init-container)<br>
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
- Create DaemonSet or Deploy DeaemonSet so that every node contain at least have 1 pod runing on it Ref [kubernetes DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#writing-a-daemonset-spec)<br>
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
### 5. secret<br>

### 6. pv & pvc <br>

### 7. network policy <br>

### 8. security context<br>

### 9. RBAC<br>

### 10. ETCD Backup<br>

### 11. Cluster Installation<br>

### 12. Cluster troubleshoot<br>

### 13. jsonpath<br>

### 14. [kubectl cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)<br>

## My CKA certificate

<p align="center">
  <img src="https://raw.githubusercontent.com/apurvabhandari/kubernetes/master/exam_prep/CKA_Apurva.png">
</p>

### Creator
Do reach out to me for any query regarding the exam.
- [Apurva Bhandari](https://www.linkedin.com/in/apurvabhandari-linux/)<br>

How to start ? <br>
- Check this [References](https://github.com/apurvabhandari/Kubernetes/blob/master/exam_prep/README.md#references)<br>
