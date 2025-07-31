# infra.new drift action

This action installs and runs the infra CLI from the `infra-new` PyPI package to perform drift detection on your Terraform infrastructure.

## Usage

```yaml
- uses: infra-new/drift-action@v1
  with:
    api-key: ${{ secrets.INFRA_API_KEY }}
    plan-file-path: 'terraform-plan.json'
    backend-dir: 'terraform'
```

## Inputs

<!-- start usage -->
| Input | Description | Required | Default |
| --- | --- | --- | --- |
| `working-directory` | Working directory to run the command in | No | `.` |
| `github-repo` | GitHub repository to pass to the drift command (defaults to current repository) | No | `${{ github.repository }}` |
| `github-ref` | GitHub ref to pass to the drift command (defaults to current ref) | No | `${{ github.ref }}` |
| `api-key` | API key for the infra command | Yes | |
| `plan-file-path` | Path to a json terraform plan. Can be created by running `terraform plan -out terraform.plan` followed by `terraform show -json > terraform-plan.json` | Yes | |
| `backend-dir` | Backend directory path | Yes | |
| `github-access-token` | GitHub access token to pass to the drift command | No | `${{ github.token }}` |
<!-- end usage -->

## Scenarios

### Basic usage

Run drift detection on the current repository with a Terraform plan:

```yaml
name: Drift Detection
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  drift-detection:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3

    - name: Terraform Init
      run: terraform init

    - name: Terraform Plan
      run: |
        terraform plan -out=terraform.plan
        terraform show -json terraform.plan > terraform-plan.json

    - name: Run Drift Detection
      uses: infra-new/drift-action@v1
      with:
        api-key: ${{ secrets.INFRA_API_KEY }}
        plan-file-path: 'terraform-plan.json'
        backend-dir: 'terraform'
```

### Different repository

Run drift detection on a different repository:

```yaml
name: Cross-Repository Drift Detection
on:
  workflow_dispatch:
    inputs:
      target_repo:
        description: 'Target repository'
        required: true
        default: 'my-org/my-infra-repo'
      target_ref:
        description: 'Target ref'
        required: true
        default: 'main'

jobs:
  drift-detection:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        repository: ${{ github.event.inputs.target_repo }}
        ref: ${{ github.event.inputs.target_ref }}
        token: ${{ secrets.GITHUB_TOKEN }}

    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v3

    - name: Terraform Init
      run: terraform init

    - name: Terraform Plan
      run: |
        terraform plan -out=terraform.plan
        terraform show -json terraform.plan > terraform-plan.json

    - name: Run Drift Detection
      uses: infra-new/drift-action@v1
      with:
        api-key: ${{ secrets.INFRA_API_KEY }}
        plan-file-path: 'terraform-plan.json'
        backend-dir: 'terraform'
        github-repo: ${{ github.event.inputs.target_repo }}
        github-ref: ${{ github.event.inputs.target_ref }}
        github-access-token: ${{ secrets.GITHUB_TOKEN }}
```

### Custom working directory

Run drift detection in a subdirectory:

```yaml
- name: Run Drift Detection
  uses: infra-new/drift-action@v1
  with:
    working-directory: './infrastructure'
    api-key: ${{ secrets.INFRA_API_KEY }}
    plan-file-path: 'terraform-plan.json'
    backend-dir: 'terraform'
```

## Permissions

When using this action with a custom `github-access-token`, ensure the token has appropriate permissions to access the target repository.

For the default `github.token`, no additional permissions are typically required for basic usage.

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE)
