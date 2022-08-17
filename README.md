# poc-k8s-zabbix

Proof of Concept for Kubernetes and Zabbix Monitoring

```
k3d cluster create \
    --config k3d-cluster.yaml

kubectl apply --filename namespace.yaml

kubectl apply --filename serviceaccount.yaml
kubectl apply --filename secret.yaml

kubectl get secret zabbix-token \
    --namespace zabbix \
    --output json \
    | jq '.data' > zabbix-token.json

jq '.["ca.crt"] | @base64d' zabbix-token.json --raw-output > ca.crt
jq '.["token"]  | @base64d' zabbix-token.json --raw-output > token
```
