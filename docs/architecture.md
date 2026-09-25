# Manual Approval Action Architecture

`tech55-infra-git-manual-approval` implements a Docker-based GitHub Action that pauses a workflow until configured approvers approve or deny through a GitHub issue comment.

## Runtime flow

1. The action reads GitHub Actions environment variables and user inputs from `action.yaml`.
2. It creates a GitHub client using `GITHUB_TOKEN` or the token passed through the `secret` input.
3. It resolves the target repository where the approval issue will be created.
4. It creates an issue with the workflow run URL, required approvers, and accepted approval/denial words.
5. It adds the custom issue body or issue body file as one or more issue comments.
6. It polls issue comments until the minimum approval count is reached or a denial is found.
7. It closes the issue and writes action outputs such as `issue-number`, `issue-url`, and `approval-status`.

## Main files

| File | Purpose |
| --- | --- |
| `action.yaml` | GitHub Action metadata, inputs, outputs, and Docker image reference. |
| `main.go` | Entrypoint, input validation, GitHub client setup, polling loop, and output handling. |
| `approval.go` | Approval issue creation, comment parsing, output writing, and approval word matching. |
| `approvers.go` | Approver resolution, including users and organization teams. |
| `approvalstatus.go` | Approval status values. |
| `constants.go` | Environment variable and default constants. |
| `Dockerfile` | Container image used by the action. |
| `Makefile` | Build, push, test, tidy, and lint commands. |

## Development workflow

Use `make test` for unit tests. Use `make build` or `make build_push` with a `VERSION` value when preparing or publishing an image.

For release work, update the Docker image reference in `action.yaml`, merge through pull request, then publish the version tags and GitHub release.

## Operational notes

- The action consumes workflow runner time while waiting for approval.
- Team approvers require a token that can read organization membership.
- GitHub App tokens expire, so approval duration must fit the token lifetime when that token type is used.
- Large approval bodies should use `issue-body-file-path` to avoid argument size limits, but very large files can hit GitHub secondary rate limits because they are split into comments.