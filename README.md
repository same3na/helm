# Same3na

## info
- Project components are separated to different charts.
- the services are in the app chart that will create the different pods for each service
- the logging chart is responsible for creating the logging services
- intall the following charts by order


## Ingress
- `helm install ingress-controller charts/ingress-controller/ -n ingress-nginx --create-namespace`

## Cert manager
- In this project Cert manager is being used to create self-signed certificates authority and certs like the one we created for internal vault communication
- install dependencies before installing the chart `helm dependency update charts/cert-manager`
- install `helm install cert-manager charts/cert-manager -n cert-manager --create-namespace`
- upgrade `helm upgrade cert-manager charts/cert-manager -n cert-manager`
- uninstall `helm uninstall cert-manager -n cert-manager`

## logging
- install dependencies before installing the chart `helm dependency update charts/logging`
- install the chart in the logging namespace `helm install logging charts/logging -n logging --create-namespace`
- uninstall the logging chart `helm uninstall logging -n logging`

## Monitoring
- install dependencies before installing the chart `helm dependency update charts/monitoring`
- install the chart in the logging namespace `helm install monitoring charts/monitoring -n monitoring --create-namespace`
- uninstall the logging chart `helm uninstall monitoring -n monitoring`

## app 
- upgrade `helm upgrade same3na-chart charts/app -f charts/app/values.prod.yaml`

## vault
- install `helm install vault charts/vault -n vault --create-namespace`
- upgrade `helm upgrade vault charts/vault -n vault`
- uninstall `helm uninstall vault -n vault`

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
- install `helm install external-secrets charts/external-secrets -n external-secrets --create-namespace`
- upgrade `helm upgrade external-secrets charts/external-secrets -n external-secrets`
- uninstall `helm uninstall external-secrets -n external-secrets`


