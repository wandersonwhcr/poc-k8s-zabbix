# poc-k8s-zabbix

Proof of Concept for Kubernetes and Zabbix Monitoring

```
k3d cluster create \
    --config k3d-cluster.yaml

kubectl apply --filename namespace.yaml

kubectl apply --filename serviceaccount.yaml
kubectl apply --filename secret.yaml
kubectl apply --filename clusterrole.yaml
kubectl apply --filename clusterrolebinding.yaml
```

```
kubectl get secret zabbix-token \
    --namespace zabbix \
    --output json \
    | jq '.data' > zabbix-token.json

jq '.["ca.crt"] | @base64d' zabbix-token.json --raw-output > ca.crt
jq '.["token"]  | @base64d' zabbix-token.json --raw-output > token

kubectl config view \
    --minify \
    --output json \
    | jq '.clusters[].cluster.server' --raw-output > kubernetes-api
```

```
KUBERNETES_API=`cat kubernetes-api`
TOKEN=`cat token`

curl "$KUBERNETES_API/api/v1/namespaces" \
    --include \
    --cacert ./ca.crt \
    --header "Authorization: Bearer $TOKEN"
```
