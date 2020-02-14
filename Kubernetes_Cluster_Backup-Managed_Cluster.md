# Kubernetes Cluster Backup - Managed Cluster
Use below Shell script to take backup of cluster where you do have backup of Master (gke, eks, aks, iks etc)

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

