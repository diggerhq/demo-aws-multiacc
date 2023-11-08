# Guide: multiple segregated AWS accounts

In this guide you will set up Digger to use completely segregated AWS accounts for Dev and Prod environments

## Prerequisites
- 2 separate Terraform projects with remote backends configured (this repo uses S3; see `backend` folder)
- 2 pairs of AWS keys

## Create 2 digger.yml files

Place `digger.prod.yml` and `digger.dev.yml` files in the root of your repo. Point `dir` to folders with terraform

```
projects:
- name: production
  dir: prod
```

## Create 2 environments in GitHub
- In your GitHub repo, go to Settings > Environments
- Press "New Environment"
- Name one "development" and another "production"

In each environment, create 2 secrets corresponding to your AWS accounts:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

## Create 2 Actions workflow files
- `.github/workflows/digger-run-dev.yml` for dev
- `.github/workflows/digger-run-prod.yml` for prod

Don't forget to change `environment` and the Rename step from Dev to Prod

```
name: Digger Dev

on:
  pull_request:
    branches: [ "main" ]
    types: [ opened, synchronize ]
  issue_comment:
    types: [created]
  workflow_dispatch:

jobs:
  plan:
    runs-on: ubuntu-latest
    environment: development                      # CHANGE HERE
    permissions:    
      contents: write      # required to merge PRs
      id-token: write      # required for workload-identity-federation
      pull-requests: write # required to post PR comments
      statuses: write      # required to validate combined PR status

    steps:
      - uses: actions/checkout@v4
      - name: Rename       # workaround until passing custom digger.yml is supported
        run: |
          mv digger.dev.yml digger.yml           # CHANGE HERE
      - name: digger run
        uses: diggerhq/digger@v0.2.0
        with:
          setup-aws: true
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
        env:
          GITHUB_CONTEXT: ${{ toJson(github) }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Verify that it works
That's it! Now you can use Digger to automate your Terraform PRs.

- Create a PR that changes terraform in one of your projects
- You should see 2 Actions jobs started
- Shortly after, a comment with plan output for the affected project will be added
- You can comment `digger apply` to apply changes
- If you do so, another Action job will start to run apply


-----------


## Running terraform locally

1. Set up S3 backend for Dev: run `terraform init` and `terraform apply` in ./backend/backend-dev
2. Plan and apply Dev: run `terraform init`, `terraform plan` and `terraform apply` in ./dev
3. Repeat 1 and 2 for Prod
