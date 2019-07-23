# Installing an Ingress controller using Nginx

## Installing Helm
1- Download script
```
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
```
2- Helm client install
```
hmod 700 get_helm.sh
./get_helm.sh
```
3- Init Helm
```
helm init
```

## Installing Tiller
1- Run the following commands to install the server-side tiller to the Kubernetes cluster with RBAC enabled
```
kubectl create serviceaccount --namespace kube-system tiller
```
2- Create a tiller Service Account
```
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 
```
3- Initialize Helm with newly-created service account
```
helm init --service-account tiller --upgrade
```
4- Checking tiller running
```
kubectl get deployments -n kube-system
```

## Deploy an application in Kubernetes
1- Deploying simple Hello-app
```
kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:1.0
```
2- Expose the hello-app Deployment as a Service 
```
kubectl expose deployment hello-app  --port=8080
```
## Deploying the NGINX Ingress Controller via Helm
