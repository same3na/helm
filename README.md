# Same3na

## info
- Project components are separated to different charts.
- the services are in the app chart that will create the different pods for each service
- the logging chart is responsible for creating the logging services
- intall the following charts by order


## Ingress
- `kustomize build --enable-helm charts/ingress-controller/. | kubectl apply -f -`

## Cert manager
- In this project Cert manager is being used to create self-signed certificates authority and certs like the one we created for internal vault communication
- `kustomize build --enable-helm charts/cert-manager/. | kubectl apply -f -`

## logging
- `kustomize build --enable-helm charts/logging/. | kubectl apply -f -`

## Monitoring
- `kustomize build --enable-helm charts/monitoring/. | kubectl apply -f -`

## app 
- upgrade `helm upgrade same3na-chart charts/app -f charts/app/values.prod.yaml`

## vault
- `kustomize build --enable-helm charts/vault/. | kubectl apply -f -`

### For vault after pods is working and unsealed I need to create policy and role for kubernetes auth
- login to the pod
- export vault url `export VAULT_ADDR=https://vault.vault.svc:8200`
- export vault token `export VAULT_TOKEN=<token here>`
- add secrets engine `vault secrets enable -path=secret -version=2 kv`
- enable kubernetes auth `vault auth enable kubernetes`
- configure kubernete auth
  ```
  vault write auth/kubernetes/config \
    token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
    disable_iss_validation=true
  ```
- create the policy in vault-policy.hcl 
  ```
  path "secret/data/*" {
    capabilities = ["read"]
  }
  ```
- write the policy `vault policy write external-secrets-policy vault-policy.hcl`

- Create Role for ESO
  ```
  vault write auth/kubernetes/role/external-secrets \
    bound_service_account_names=external-secrets \
    bound_service_account_namespaces=external-secrets \
    policies=default,external-secrets-policy \
    ttl=24h
  ```

## external-secrets
- `kustomize build --enable-helm charts/external-secrets/. | kubectl apply -f -`

## argoCD
- `kustomize build --enable-helm charts/argoCD/. | kubectl apply -f -`
