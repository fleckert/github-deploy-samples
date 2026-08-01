A repo for code samples

# open Git repo online

Notes:

- This is for `zsh`
- This is for `remote`s named `origin`
- This does not work when a port is specified
- This is a alternative to `gh browse`... in case you don't have the GitHub Cli installed.
- This works for repositories in https://bitbucket.org/ as well...

Checking out this repo via ssh protocol results in 

```bash
> git remote -v 
origin  git@github.com:fleckert/github-deploy-samples.git (fetch)
origin  git@github.com:fleckert/github-deploy-samples.git (push)
```

Checking out this repo via https protocol results in 

```bash
> git remote -v
origin  https://github.com/fleckert/github-deploy-samples.git (fetch)
origin  https://github.com/fleckert/github-deploy-samples.git (push)
```

and resolving the Git server url with

``` bash
git remote -v | grep '(push)' | sed 's/origin\t//g' | sed 's/\.git (push)//' | sed 's/:/\//' | sed 's/git@/https:\/\//g' | sed 's/https\/\/\//https:\/\//g'
```
|Input|Output|
|-|-|
|origin https://github.com/fleckert/github-deploy-samples.git (push) | https://github.com/fleckert/github-deploy-samples|
|origin git@github.com:fleckert/github-deploy-samples.git (push)     | https://github.com/fleckert/github-deploy-samples||

``` bash
# add this to ~/.zshrc
#
# This command converts a Git remote URL (SSH or HTTPS) into a browser-accessible HTTPS URL.
# see ./.git/config

# git remote -v                    - Lists all remote repositories with their URLs
# grep '(push)'                    - Filters to show only the push URL line
# sed 's/origin\t//g'              - Removes "origin" and the tab character
# sed 's/\.git (push)//'           - Removes ".git (push)" from the end
# sed 's/:/\//'                    - Replaces the first colon with a slash (converts SSH format github.com:user/repo to github.com/user/repo)
# sed 's/git@/https:\/\//g'        - Replaces git@ with https://
# sed 's/https\/\/\//https:\/\//g' - Fixes any accidental triple slashes to double slashes

alias opengit='git remote -v &>/dev/null && open $(git remote -v | grep '\''(push)'\'' | sed '\''s/origin\t//g'\'' | sed '\''s/\.git (push)//'\'' | sed '\''s/:/\//'\'' | sed '\''s/git@/https:\/\//g'\'' | sed '\''s/https\/\/\//https:\/\//g'\'')'
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

The point I am trying to make... the User Assigned Managed Identity does NOT have to be used on the GitHub Runner, it can be used like any other Microsoft Entra ID application.


The GitHub Action [azure/login](https://github.com/azure/login)  is the default way how to log in, but the Azure CLI or Azure Powershell + GitHub native approach works as well.

```bash
az login
    --service-principal
    --tenant   <tenant_id>
    --username <client_id>
    --federated-token "$(
        curl --silent --header "Authorization: Bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN"
        "$ACTIONS_ID_TOKEN_REQUEST_URL&audience=api://AzureADTokenExchange" | jq --raw-output '.value'
      )"
```

```powershell
Connect-AzAccount
    -ServicePrincipal
    -Tenant        <tenant_id>
    -ApplicationId <client_id>
    -FederatedToken $(
        Invoke-RestMethod -Headers @{Authorization = "Bearer $env:ACTIONS_ID_TOKEN_REQUEST_TOKEN"} 
        -Uri "$env:ACTIONS_ID_TOKEN_REQUEST_URL&audience=api://AzureADTokenExchange"
     ).value

```

# .github/workflows/git-checkout.yml

Checking out the current repository using `git` commands.

This might be beneficial if the `git` commands work well enough and you don't want to use external code i.e. [actions/checkout](https://github.com/actions/checkout).

``` bash
SERVER_HOST=$(echo "${{ github.server_url }}" | sed 's|https://||')
git clone https://x-access-token:${{ secrets.GITHUB_TOKEN }}@${SERVER_HOST}/${{ github.repository }}.git .
git switch ${{ github.event.pull_request.head.ref || github.ref_name }}
echo "SHA $(git rev-parse HEAD)"
```
# .github/workflows/github-secrets.yml
If you want to dump the value of a GitHub Secret, see [github-secrets.yml](./.github/workflows/github-secrets.yml)

```
echo "${{ secrets.MY_SECRET_NAME }}" | sed 's/./& /g
```

the value shows up with blanks between the characters of the secret to avoid masking

