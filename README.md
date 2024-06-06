# gha-release-flow

This repository provides a composite GitHub Action that can be reused in workflows to support the Release Flow branching strategy.  It also shares an example workflow to support the use of Release Flow with GitHub Actions.  The need for this was written with Terraform as the use case.

The action requires a GitHub App to be created and associated with the repository.  The app requires 'read' permissions for "Environments" and "Variables".  The app-id is safe to store as repository variable.  The private-key should be stored as a repository secret.

The result of the action is a strategy matrix to that is used to filter the environments to only those that are aligned to the release branch.  This allows for a single workflow to be used for all releases/environments.

## Inputs

* app-id
  * description: GitHub App "app id" with 'read' permissions for "Environments" and "Variables".  Safe to store as repository "variable".
  * required: true
* private-key
  * description: GitHub App "private key" associated with "app id".  Safe to store as repository "secret".
  * required: true
* filter-on
  * description: Environment variable to filter on.  Defaults to "RELEASE".  Safe to store as repository "variable".
  * required: false
* filter-by
  * description: Value to filter environment variable by.  Defaults to 'github.ref_name'.
  * required: false

## Outputs

* matrix
  * description: Strategy matrix listing all environments matching "filter-on" and "filter-by".  Use in subsequent job as `fromJson(needs.<job_name>.outputs.matrix' and within 'environment' as 'matrix.environment'.

## Usage

```yaml
jobs:

  release_matrix:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.create-matrix.outputs.matrix }}
    env:
      repo: ${{ github.repository }}
    steps:
      - id: create-matrix
        name: Create matrix
        uses: dustindortch/github-actions-release-flow@v1
        with:
          app-id: ${{ vars.FILTER_APP_ID }}
          private-key: ${{ secrets.FILTER_APP_PRIVATE_KEY }}
          filter-by: ${{ github.ref_name }}

  run_workflow:
    needs: release_matrix
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include: ${{ needs.release_matrix.outputs.matrix }}
    environment: ${{ matrix.environment }}
    steps:
      - id: run-workflow
        name: Run workflow
        run: |
          set -eux
          echo "Running workflow for ${{ matrix.environment }}"
```
