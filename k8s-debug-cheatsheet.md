## Bad Pods
```
kubectl get pods --all-namespaces -o wide | egrep -v ‘Running|Completed|Error|BackOff|OOMKilled’
```
## Autoscaler State ConfigMap
```
kubectl -n kube-system describe configmap/cluster-autoscaler-status
```
## Autoscaler logs (all pods)
```
kubectl logtail --namespace=kube-system kube-cluster-autoscaler --since=10m
```
## Deleting a pod that’s stuck in “Terminating” state for more than 5 minutes
```
kubectl delete pod <pod-name> --force --grace-period=0
```
## Cluster spot instances per node
```
kubectl get nodes -L lifecycle-status -L 'failure-domain.beta.kubernetes.io/zone' -L beta.kubernetes.io/instance-type -L aws.amazon.com/spot
```
