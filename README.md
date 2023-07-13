# overmind actions

Github Actions to run [Overmind](https://overmind.tech/) Terraform Impact Analysis on PRs

# Usage

The `install` action installs the [`ovm-cli`](https://github.com/overmindtech/ovm-cli).

```
- uses: overmindtech/actions/install-cli@main
  with:
    version: latest # request a specific version for install. Defaults to `latest`.
    github-token: ${{ secrets.GITHUB_TOKEN }} # avoid API limits
    github-api-url: # Use http(s)://[hostname]/api/v3 to access the API for GitHub Enterprise Server
```

The `submit-plan` action takes a JSON-formatted terraform plan, creates a Overmind Change for it, and runs Impact Analysis.

```
- uses: overmindtech/actions/submit-plan@main
  id: submit-plan
  with:
    ovm-api-key: ${{ secrets.OVM_TOKEN }}
    plan-json: ./tfplan.json
```

## Complete example

Copy this workflow to `.github/workflows/overmind.yml` to run `terraform init`, `terraform plan` and submit the planned changes to Overmind.

> Note: This example does not include any configuration to allow terraform access to your infrastructure.

```
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
      - uses: actions/checkout@v3

      - uses: hashicorp/setup-terraform@v2
        with:
          terraform_wrapper: false

      - name: Terraform Init
        id: init
        shell: bash
        run: terraform init -input=false

      - name: Terraform Validate
        id: validate
        run: terraform validate -no-color

      - name: Terraform Plan
        id: plan
        run: |
          set -o pipefail -ex
          terraform plan -no-color -input=false -out tfplan 2>&1 \
            | tee terraform_log
          terraform show -json tfplan > tfplan.json

      - uses: overmindtech/actions/install-cli@main

      - uses: overmindtech/actions/submit-plan@main
        id: submit-plan
        with:
          ovm-api-key: ${{ secrets.OVM_TOKEN }}
          plan-json: ./tfplan.json

      - name: post change url as sticky comment
        uses: marocchino/sticky-pull-request-comment@v2
        with:
          header: change
          message: |
            See the blast radius of your planned changes at [Overmind](${{ steps.submit-plan.outputs.change-url }})
```

# Development

Install [nektos/act](https://github.com/nektos/act) and run

```
gh act pull_request -s GITHUB_TOKEN="$(gh auth token)" -s OVM_API_KEY="${OVM_API_KEY}"
```

to try out the `selftest` action locally. It's much faster than commit/push.
