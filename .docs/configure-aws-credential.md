# configure-aws-credential

This GitHub Workflow:
Configure your AWS credentials and region environment variables for use in other GitHub Actions. This action implements the AWS SDK credential resolution chain and exports environment variables for your other Actions to use. Environment variable exports are detected by both the AWS SDKs and the AWS CLI for AWS API calls

# Usage

- For any armor repositories where we are deploying to aws can use this workflow to assume role to shared service and then assume role to required environment(dev,stage,prod).
- The armor aws shared service account uses OpenID Connect (OIDC) which allows your GitHub Actions workflows to access resources in Amazon Web Services (AWS), without needing to store the AWS credentials as long-lived GitHub secrets.

## Create GitHub Workflow

```yaml
name: 'cd'

on:
  push:
    branches: [ "master", "main" ]
  pull_request:

permissions:
  id-token: write
  contents: read

jobs:

  cd:
    - name: Assume Role
      uses: armor/actions/.github/workflows/configure-aws-credentials.yaml@main
      with:
        AWS_SHARED_ACCOUNT_ID: arn:aws:iam::${{ env.awsSharedAccountId }}
        AWS_ACCOUNT_ID: ${{ env.awsAccountId }}
        AWS_REGION: ${{ env.region }}
        GITHUB_JOB: ${{ env.githubJob }}
        GITHUB_RUN_ATTEMPT: ${{ env.githubRunAttempt }}
        GITHUB_REPO_NAME: ${{ inputs.repoName }}

    - name: Sts GetCallerIdentity
      if: success()
      run: |
        aws sts get-caller-identity

    - name: Deploy
      run: sls deploy ##Deploy serverless or deploy to aws s3 
```
