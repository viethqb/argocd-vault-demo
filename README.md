```bash
kind create cluster --name dev --config kind-config.yaml 
```

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm search repo hashicorp/vault
helm upgrade --install vault hashicorp/vault -n vault --create-namespace --create-namespace --version 0.28.1 --debug

kubectl -n vault delete pvc --all
kubectl exec --stdin=true --tty=true vault-0 -n vault -- vault status
kubectl exec --stdin=true --tty=true vault-0 -n vault -- vault operator init

# Unseal Key 1: ody+EXrvOmSnTfTCfnd5e7Nn0ZXQWrOYX4FyDJVZO8w6
# Unseal Key 2: CBsqvavOY0TqfbZmSup5NWXkP0b/2dWQ0gnkkFKRLiPQ
# Unseal Key 3: P/YemP4nWGHW+40vIqYwKifj0uk8wT2bqL3SJ2lUBYX+
# Unseal Key 4: D1Y90a9xAYvyi1dFHMiAuLu+yeA+mIt564CJspiFLAvD
# Unseal Key 5: I4nZdXo41+TV6QgMtdcO7cANYZJ+Y8+pCZS8GYIPMnGq

# Initial Root Token: hvs.Uj8X0oEIUkKhh4TK7lvPa48P

kubectl exec --stdin=true --tty=true vault-0 -n vault -- vault operator unseal
```

```bash
k -n vault exec -it vault-0 sh
vault login hvs.Uj8X0oEIUkKhh4TK7lvPa48P 
vault token create

# enable key-value engine
vault secrets enable kv-v2
# add the password to path kv-v2/argocd
vault kv put kv-v2/lakehouse/minio user="admin" password="password" endpoint="minio.minio.svc.cluster.local:9000" http-endpoint="http://minio.minio.svc.cluster.local:9000"
vault kv put kv-v2/lakehouse/metastore-postgres user="admin" password="admin" database="metastore_db" postgres-password="admin" jdbc-url="jdbc:postgresql://metastore-db-postgresql-hl:5432/metastore_db"
# add a policy to read the previously created secret
vault policy write minio - <<EOF
path "kv-v2/data/lakehouse/minio" {
  capabilities = ["read"]
}
EOF
vault policy write metastore-postgres - <<EOF
path "kv-v2/data/lakehouse/metastore-postgres" {
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