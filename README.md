# ParentTV — Org-wide GitHub Actions

This repository holds **org-wide reusable GitHub Actions workflows** for the
[ParentTV](https://github.com/ParentTV) organization. Workflows defined under
[`.github/workflows/`](.github/workflows/) can be called by any repo in the org.

## `deploy-ecs.yml`

A reusable workflow that builds a Docker image, pushes it to Amazon ECR, and
deploys it to **ECS Fargate**. It is used by all six consolidated ECS services:

- `identity-service`
- `notifications-service`
- `commerce-service`
- `catalogue-service`
- `worker-service`
- `analytics-service`

This workflow **replaces the legacy `make blue` / `make green` deployment
flow** from the EKS era.

### Authentication

Authentication uses **GitHub OIDC federation to AWS** — there are **no
long-lived AWS credentials stored anywhere**. At deploy time, GitHub Actions
exchanges a short-lived OIDC token for temporary AWS credentials by assuming a
dedicated IAM role.

- **IAM role:** `arn:aws:iam::571604588115:role/parenttv-github-actions-deploy-role`

The role ARN is provided to the workflow through the org-level secret
`AWS_DEPLOY_ROLE_ARN`, inherited automatically by every service repo.

### Usage

Each service repo references the reusable workflow like so:

```yaml
jobs:
  deploy:
    uses: ParentTV/.github/.github/workflows/deploy-ecs.yml@main
    with:
      service-name: identity-service
      ecr-repository: identity-service
      ecs-cluster: parenttv-prod
      ecs-service: identity-service
      task-definition: .aws/identity-service-task-definition.json
      container-name: identity-service
    secrets:
      AWS_DEPLOY_ROLE_ARN: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
```

See [`examples/deploy-identity-service.yml`](examples/deploy-identity-service.yml)
for a complete calling workflow.

> **Note on the nested path:** `ParentTV/.github/.github/workflows/deploy-ecs.yml`
> is correct. GitHub's org-wide convention places reusable workflows inside the
> `.github/` folder of the special `.github` repository, producing the doubled
> path segment.
