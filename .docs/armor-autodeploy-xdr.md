# armor-autodeploy-xdr

This GitHub Workflow:
- when a PR is created, will run `terragrunt plan` and create a PR comment with the output of the plan
- when commit is made to master or main branch, will run `terragrunt apply`

# Usage

The owner of the Armor XDR will need to
- For each Armor XDR environment, create a service principal that can update that environment.
- Create a GitHub deploy key for reading the `infrastructure-modules` repository.
- Create a GitHub Workflow in the `infrastructure-live` repository that will in turn call this Workflow.

## Create a service principal

This service principal should have all the necessary permissions to update all the resources in an Armor XDR.
The documentation in `infrastructure-live` lists all the permissions.

## Create GitHub Deploy Key

This GitHub Deploy Key is necessary to pull in the `infrastructure-modules` repository. The [GitHub documentation](
https://docs.github.com/en/authentication/connecting-to-github-with-ssh/managing-deploy-keys#set-up-deploy-keys)
details how to do it. We list a summary here:

1. Create SSH key pair
2. Upload public key to the `infrastructure-modules` Deploy Keys page on GitHub
3. The private key will be used in the subsequent steps

## Create GitHub Workflow

### Step 1 - Add secrets

The GitHub Workflow will require some secrets.
- Authentication credentials (for the service principal) for each of the environments that are deployed
- The private key of the Deploy Key

**Add these secrets to GitHub Secrets of the `infrastructure-live` repository**.

### Step 2 - Create workflow file

Next **create a file `xdr.yaml` in the `.github/workflows` directory** of the `infrastructure-live` repository.

> **Note**
> It doesn't have to be `xdr.yaml`. It can be any yaml file.
> But it must be in the `.github/workflows` directory.

The contents of the `xdr.yaml` should look like this:


```yaml
name: 'XDR'

on:
  push:
    branches: [ "master", "main" ]
  pull_request:

permissions:
  id-token: write
  contents: write
  pull-requests: write

jobs:

  armor-auto-deploy-xdr:
    uses: armor/actions/.github/workflows/armor-autodeploy-xdr.yaml@main
    secrets:
      XDR_CREDENTIALS: '{
        "azure": {
            "demo": {
              "location": "southeastasia",
              "clientId": "${{ secrets.AZURE_DEMO_CLIENTID}}",
              "clientSecret": "${{ secrets.AZURE_DEMO_CLIENTSECRET }}"
            }
        }
      }'
      DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

- `DEPLOY_KEY` should be replaced by the secret name of the GitHub Deploy Key (see [above](#create-github-deploy-key))
- `"azure"` is the provider name. We currently only support `"azure"`.
- `"demo"` should be replaced by the environment name. This is the same as the directory name under `azure` of Armor's
  `infrastructure-live`.
- `southeastasia` should be replaced by the location / region of the subscription (for Azure)
- `AZURE_DEMO_CLIENTID` and `AZURE_DEMO_CLIENTSECRET` should be replaced by the secret name of the credentials.

If the XDR infrastructure has more than 1 environment, say "staging" and "production", then the `XDR_CREDENTIALS` string
should look like:

```yaml
      XDR_CREDENTIALS: '{
        "azure": {
            "staging": {
              "location": "southeastasia",
              "clientId": "${{ secrets.SECRET_1 }}",
              "clientSecret": "${{ secrets.SECRET_2 }}"
            },
            "prod": {
              "location": "southeastasia",
              "clientId": "${{ secrets.SECRET_3 }}",
              "clientSecret": "${{ secrets.SECRET_4 }}"
            }
        }
      }'
```

> **Note**
> Remember the environment names must correspond to the directory names under `azure` of `infrastructure-live`.
