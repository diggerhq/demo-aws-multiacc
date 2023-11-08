# demo-aws-multiacc

## Running locally

1. Set up S3 backend for Dev: run `terraform init` and `terraform apply` in ./backend/backend-dev
2. Plan and apply Dev: run `terraform init`, `terraform plan` and `terraform apply` in ./dev
3. Repeat 1 and 2 for Prod
