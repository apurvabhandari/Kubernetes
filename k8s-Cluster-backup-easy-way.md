# Kubernetes Cluster Backup - Easy Way
## Resource Configuration Backup
### kube-apiserver
```
kubectl get all --all-namespaces -o yaml > all-services.yaml
```

## ETCD Cluster Backup
![img](https://raw.githubusercontent.com/apurvabhandari/kubernetes/master/etcd.png) <br>
### Backup of data directory
```
etcd.services
..
..
--data-dir=/var/lib/etcd
```
### OR

 ### Backup by ETCD backup command line 
 1- Take snapshot with certs
```
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/etcd/ca.crt \
--cert=/etc/etcd/etcd-server.crt \
--key=/etc/etcd/etcd-server.key
```
2- Check status
```
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```
### Restore ETCD
1- Stop kube-apiserver
```
service kube-apiserver stop
```
2- Restor snapshot with new token-id as ``` etcd-cluster-1 ``` 
```
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
--data-dir /var/lib/etcd-backup \
--initial-cluster master-1=https://10.10.10.90:2380, master-2=https://10.10.10.91:2380 \
--initial-token-cluster etcd-cluster-1 \
--initial-advertise-peer-url https://${INTERNAL_IP}:2380
```
3- Change new token and data directory in ```etcd.service``` file  and check data-dir
```
etcd.service
--initial-cluster-token etcd-cluster-1
--data-dir=/var/lib/etcd
```
4- Reload daemon, restart etcd, start kube-apiserver
```
systemctl daemon-relod
service etcd restart
service kube-apiserver start
```
## Backup for fully-managed Cluster
Consider you do not have access to ETCT or Master node<br>
Following shell script you can use to take backup of running resources in your cluster. You can restore that by simply applying that files on your new cluster or existing cluster (gke, eks, aks, iks etc)
```
i=$((0))
for n in $(kubectl get -o=custom-columns=NAMESPACE:.metadata.namespace,KIND:.kind,NAME:.metadata.name pv,pvc,configmap,ingress,service,secret,deployment,statefulset,hpa,job,cronjob --all-namespaces | grep -v 'secrets/default-token')
do
	if (( $i < 1 )); then
		namespace=$n
		i=$(($i+1))
		if [[ "$namespace" == "PersistentVolume" ]]; then
			kind=$n
			i=$(($i+1))
		fi
	elif (( $i < 2 )); then
		kind=$n
		i=$(($i+1))
	elif (( $i < 3 )); then
		name=$n
		i=$((0))
		echo "saving ${namespace} ${kind} ${name}"
		if [[ "$namespace" != "NAMESPACE" ]]; then
			mkdir -p $namespace
			kubectl get $kind -o=yaml $name -n $namespace > $namespace/$kind.$name.yaml
		fi
	fi
done
```
