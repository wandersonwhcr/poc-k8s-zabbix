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

KUBERNETES_API=`cat kubernetes-api`
TOKEN=`cat token`
```

```
curl "$KUBERNETES_API/api/v1/nodes" \
    --fail --silent \
    --cacert ./ca.crt \
    --header "Authorization: Bearer $TOKEN" \
    | jq '.items[].metadata.name' \
    | sort --numeric-sort

"k3d-cluster-server-0"
"k3d-cluster-agent-0"
"k3d-cluster-agent-1"
"k3d-cluster-agent-2"
```

```
curl "$KUBERNETES_API/api/v1/namespaces" \
    --fail --silent \
    --cacert ./ca.crt \
    --header "Authorization: Bearer $TOKEN" \
    | jq '.items[].metadata.name' \
    | sort --numeric-sort

"default"
"kube-node-lease"
"kube-public"
"kube-system"
"zabbix"
```

```
curl "$KUBERNETES_API/api/v1/pods" \
    --fail --silent \
    --cacert ./ca.crt \
    --header "Authorization: Bearer $TOKEN" \
    | jq '.items[].metadata' \
    | jq --slurp 'sort_by(.namespace, .name) | .[]' \
    | jq '{ "namespace": .namespace, "name": .name }' --compact-output

{"namespace":"kube-system","name":"coredns-d76bd69b-hchvr"}
{"namespace":"kube-system","name":"helm-install-traefik-crd-9s72x"}
{"namespace":"kube-system","name":"helm-install-traefik-zplgf"}
{"namespace":"kube-system","name":"local-path-provisioner-6c79684f77-q57k7"}
{"namespace":"kube-system","name":"metrics-server-7cd5fcb6b7-928dq"}
{"namespace":"kube-system","name":"svclb-traefik-5x6zl"}
{"namespace":"kube-system","name":"svclb-traefik-b94vc"}
{"namespace":"kube-system","name":"svclb-traefik-d7x2v"}
{"namespace":"kube-system","name":"svclb-traefik-dxpgd"}
{"namespace":"kube-system","name":"traefik-df4ff85d6-d5k2b"}
```

```
curl "$KUBERNETES_API/apis/apps/v1/replicasets" \
    --fail --silent --show-error \
    --cacert ./ca.crt \
    --header "Authorization: Bearer $TOKEN" \
    | jq '.items[].metadata' \
    | jq --slurp 'sort_by(.namespace, .name) | .[]' \
    | jq '{ "namespace": .namespace, "name": .name }' --compact-output

{"namespace":"kube-system","name":"coredns-d76bd69b"}
{"namespace":"kube-system","name":"local-path-provisioner-6c79684f77"}
{"namespace":"kube-system","name":"metrics-server-7cd5fcb6b7"}
{"namespace":"kube-system","name":"traefik-df4ff85d6"}
```
