A repo for code samples

# open Git repo online

I am using macOS right now, so the example is for zsh

``` bash
# add this to .zshrc

alias opengit='open $(git remote -v | grep push | sed "s/.*@\([^:]*\):\([^.]*\).*/https:\/\/\1\/\2/")'
```

and within a git repo

``` bash
opengit
```

will open the online Git repo

# .github/workflows/deploy-with-managed-identity.yml

Logging into Azure with a federated credentials setup using an User Assigned Managed Identity works as good as an Microsoft Entra ID Application.

A possible benefit may be due to processes in an enterprise setup.
- You have permissions within the Azure subscription.
- You have to go through an 'enterprise IAM' procedure to get an Microsoft Entra ID Application.

Here, creating an User Assigned Managed Identity might be a quick way to provide the Azure context for GitHub workflows.

The point I am trying to make... the User Assigned Managed Identity does NOT have to be used on the GitHUb Runner, it can be used like any other Microsoft Entra ID application.


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


# .github/workflows/git-checkout.yml

Checking out the current repository using
- [actions/checkout](https://github.com/actions/checkout)
- using `git` commands

This might be beneficial if the `git` commands work well enough and you don't want to use external code.


# .github/workflows/github-secrets.yml
If you want to dump the value of a GitHub Secret, see [github-secrets.yml](./.github/workflows/github-secrets.yml)

```
echo "${{ secrets.MY_SECRET_NAME }}" | sed 's/./& /g
```

the value shows up woth blanks between the characters of the secret to avoid masking

