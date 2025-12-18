A repo for code samples

# .github/workflows/deploy-with-managed-identity.yml

Logging into Azure with a federated credentials setup using an User Assigned Managed Identity works as good as an Microsoft Entra ID Application.

A possible benefit may be due to processes in an enterprise setup.
- You have permissions within the Azure subscription.
- You have to go through an 'enterprise IAM' procedure to get an Microsoft Entra ID Application.

Here, creating an User Assigned Managed Identity might be a quick way to provide the Azure context for GitHub workflows.


The GitHub Action [azure/login](https://github.com/azure/login)  is the default way how to log in, but the Azure CLI + GitHub native approach works as well.

```bash
az login
    --service-principal
    --allow-no-subscriptions
    --username <client_id>
    --tenant   <tenant_id>
    --federated-token "$(
        curl --silent --header "Authorization: Bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN"
        "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=api://AzureADTokenExchange" | jq --raw-output '.value'
      )"
```
