```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm search repo hashicorp/vault
helm install vault hashicorp/vault -n vault --create-namespace --values vault-server-values.yaml --create-namespace --version 0.28.1 --debug
```

```bash
k -n vault exec -it vault-0 sh
vault login root 
vault token create
#hvs.D72pVl1lFSIqwVmDOCZnEB5X

# enable key-value engine
vault secrets enable kv-v2
# add the password to path kv-v2/argocd
vault kv put kv-v2/argocd admin="admin" password="argocd"
# add a policy to read the previously created secret
vault policy write argocd - <<EOF
path "kv-v2/data/argocd" {
  capabilities = ["read"]
}
EOF
```

```bash
kubectl create ns argocd
kubectl apply -f secret.yaml -n argocd
kubectl apply -f cmp-plugin.yaml -n argocd
```

```bash
helm upgrade --install argocd argo/argo-cd -f argocd-values.yaml --set server.service.type=NodePort --namespace argocd --create-namespace --version 7.4.3 --debug

ARGOPW=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
echo $ARGOPW
```

```bash
k apply -f avp-argocd.yaml
k apply -f minio-argocd.yaml 
```