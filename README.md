# poc-k8s-zabbix

Proof of Concept for Kubernetes and Zabbix Monitoring

```
k3d cluster create \
    --config k3d-cluster.yaml

kubectl apply --filename namespace.yaml

kubectl apply --filename serviceaccount.yaml
kubectl apply --filename secret.yaml
```
