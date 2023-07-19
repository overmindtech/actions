# Overmind Actions

Use [Overmind](https://overmind.tech/) to calculate the blast radius of your Terraform pull requests.

![blast radius preview](./doc/blast_radius.png)

# Usage

The `install` action installs the [`ovm-cli`](https://github.com/overmindtech/ovm-cli).

```yaml
- uses: overmindtech/actions/install-cli@main
  with:
    version: latest # Request a specific version for install. Defaults to `latest`.
    github-token: ${{ secrets.GITHUB_TOKEN }} # Avoid API limits (optional)
    github-api-url: https://ghe.company.com/api/v3 # API for GitHub Enterprise Server (optional)
```

The `submit-plan` action takes a JSON-formatted terraform plan, creates a Overmind Change for it, and runs Impact Analysis.

```yaml
- uses: overmindtech/actions/submit-plan@main
  id: submit-plan
  with:
    ovm-api-key: ${{ secrets.OVM_TOKEN }} # Generated within Overmind
    plan-json: ./tfplan.json # Location of the plan in JSON format
```

## Complete example

Copy this workflow to `.github/workflows/overmind.yml` to run `terraform init`, `terraform plan` and submit the planned changes to Overmind.

> Note: This example does not include any configuration to allow terraform access to your infrastructure.

```yaml
name: Terraform Validation
on: [pull_request]

jobs:
  plan:
    runs-on: ubuntu-latest
    permissions:
      contents: read # required for checkout
      pull-requests: write # create/update a comment
    concurrency:
      group: tfstate # avoid running more than one job at the same time

    steps:
      # Checkout your code
      - uses: actions/checkout@v3

      # Set up Terraform
      - uses: hashicorp/setup-terraform@v2
        with:
          terraform_wrapper: false
      - name: Terraform Init
        id: init
        shell: bash
        run: terraform init -input=false

      # Run Terraform plan. Note that these commands will allow terraform to
      # log nicely and also create a plan JSON file
      - name: Terraform Plan
        id: plan
        run: |
          set -o pipefail -ex
          terraform plan -no-color -input=false -out tfplan 2>&1 \
            | tee terraform_log
          terraform show -json tfplan > tfplan.json

      # Install the Overmind CLI
      - uses: overmindtech/actions/install-cli@main

      # Submit the plan. This will add a comment with the blast radius
      - uses: overmindtech/actions/submit-plan@main
        id: submit-plan
        with:
          ovm-api-key: ${{ secrets.OVM_TOKEN }}
          plan-json: ./tfplan.json
```

# Development

Install [nektos/act](https://github.com/nektos/act) and run

```
gh act pull_request -s GITHUB_TOKEN="$(gh auth token)" -s OVM_API_KEY="${OVM_API_KEY}"
```

to try out the `selftest` action locally. It's much faster than commit/push.
