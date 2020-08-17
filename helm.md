# Helm
## Helm basic
`source <(helm completion bash)`

`helm create newchart`

`helm lint .`

`helm ls`

`helm list`

`heml install deploy_name newchart`

`helm repo add repo_name repo_URL`

`helm install name/deployment --version number`

When dependency in chart Chart.yaml 
`helm dep list newchart`

while building or adding Chart.yaml check the ver of dep
build with dep
`helm dep build newchart`


update dep with chart
`helm dep update newchart`

history
`helm history newchart`

upgrade deployment
`helm updagrade deploy_name newchart`

