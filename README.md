# Start Runner for Github Actions

This action start a Github Runner in EC2 machine on AWS

## Inputs

- `github-token`: The Github Personal Access Token (required)
- `instance-type`: The EC2 instance type to launch (default: `t2.medium`)
- `ami-id`: The EC2 AMI to use (fot testing, do not set it !)

AWS credentials (optional):

- `aws-access-key-id`
- `aws-secret-access-key`
- `aws-session-token`

AWS credentials can be set in a previous step or passed to the action. If credentials are passed, previously set credentials are ignored

## Outputs

- `runner-label`: The label of the created runner
- `instance-id`: The id of the created instance

## Usage

```yaml
jobs:
  start-runner:
    name: Start self-hosted EC2 runner
    runs-on: ubuntu-latest
    outputs:
      label: ${{ steps.start.outputs.runner-label }}
      id: ${{ steps.start.outputs.instance-id }}
    steps:
      - uses: pr-bob-actions/start-runner@main
        id: start
        with:
          github-token: ${{ secrets.GH_TOKEN }}
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

	job1:
		needs: [start-runner] # Don't forget this line
		runs-on: ${{ needs.start-runner.outputs.label }}
		steps:
			- run: echo do stuff

	job2:
		needs: [start-runner] # Don't forget this line
		runs-on: ${{ needs.start-runner.outputs.label }}
		steps:
			- run: echo do stuff

	# Don't forget to stop the runner !
  stop-runner:
    name: Stop self-hosted EC2 runner
    needs: [start-runner, job1,job2] # Adapt as needed
    runs-on: ubuntu-latest
    if: ${{ always() }} # Don't forget this line
    steps:
      - uses: pr-bob-actions/stop-runner@main
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}
          github-token: ${{ secrets.GH_TOKEN }}
          runner-label: ${{ needs.start-runner.outputs.label }}
          instance-id: ${{ needs.start-runner.outputs.id }}

```
