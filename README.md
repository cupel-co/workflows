# GitHub Workflows
[![Integrate](https://github.com/cupel-co/workflows/actions/workflows/integrate.yml/badge.svg?branch=main)](https://github.com/cupel-co/workflows/actions/workflows/integrate.yml?query=branch%3Amain)

Reusable workflows for GitHub

## Workflows
### Pull Request
#### Notify
Workflow: [pull-request.notify.yml](.github/workflows/pull-request.notify.yml)

Send a notification to the specified google chat

##### Secrets
| Name                      | Description                                 | Required |
|---------------------------|---------------------------------------------|----------|
| `google-chat-webhook-url` | The URL for sending messages to Google Chat | true     |

##### Example
```yaml
jobs:
  pull-request:
    name: Pull Request
    uses: cupel-co/workflows/.github/workflows/pull-request.notify.yml@vX.X.X
    with:
      google-chat-webhook-url: "${{ secrets.google-chat-webhook-url }}"
```

#### Update
Workflow: [pull-request.update.yml](.github/workflows/pull-request.update.yml)

Update the PR description

##### Supported Values
| Name                      | Description                                                       |
|---------------------------|-------------------------------------------------------------------|
| `{{BRANCH_NAME}}`         | The name of the branch                                            |
| `{{ENVIRONMENT}}`         | The environment                                                   |
| `{{ISSUE_NUMBER}}`        | The number of the issue                                           |
| `{{ISSUE_TITLE}}`         | The title of the issue                                            |
| `{{PULL_REQUEST_NUMBER}}` | The number of the pull request                                    |
| `{{PULL_REQUEST_TITLE}}`  | The pull request title                                            |
| `{{REPOSITORY}}`          | The name of the repository. This includes the the owner name too. | 

##### Inputs
| Name          | Description          | Required | Default Value |
|---------------|----------------------|----------|---------------|
| `environment` | The environment name | true     |               |

##### Secrets
| Name           | Description                                            | Required |
|----------------|--------------------------------------------------------|----------|
| `github.token` | GitHube token. Needs `pull-requests: write` permission | true     |

##### Outputs
| Name          | Description                                       |
|---------------|---------------------------------------------------|
| `description` | The pull request description with replaced values |

##### Example
```yaml
jobs:
  pull-request:
    name: Pull Request
    uses: cupel-co/workflows/.github/workflows/update-pull-request.yml@vX.X.X
    permissions:
      issues: read
      pull-requests: write
      repository-projects: read
    with:
      environment: Preview
```

### Release
#### Create
Workflow: [release.create.yml](.github/workflows/release.create.yml)

Tag, create a GitHub release and notify.

##### Inputs
| Name  | Description   | Required | Default |
|-------|---------------|----------|---------|
| `tag` | The tag value | true     |         |

##### Secrets
| Name              | Description                                            | Required |
|-------------------|--------------------------------------------------------|----------|
| `ssh-private-key` | The private SSH key used for signing the Git tag       | true     |
| `ssh-public-key`  | The public SSH key used for signing the Git tag        | true     |
| `token`           | Token for github interaction                           | true     |
| `user-email`      | Email of the user the SSH signing key is registered to | true     |
| `user-name`       | Name of the user the SSH signing key is registered to  | true     |

##### Example
```yaml
jobs:
  release:
    name: Release
    uses: cupel-co/workflows/.github/workflows/release.create.yml@vX.X.X
    permissions:
      contents: write
    with:
      tag: "v0.0.1"
    secrets:
      ssh-private-key: ${{ secrets.GH_SIGNING_SSH_PRIVATE_KEY }}
      ssh-public-key: ${{ secrets.GH_SIGNING_SSH_PUBLIC_KEY }}
      token: ${{ github.token }}
      user-email: ${{ secrets.GH_SIGNING_SSH_USER_EMAIL }}
      user-name: ${{ secrets.GH_SIGNING_SSH_USER_NAME }}
```

#### Notify
Workflow: [release.notify.yml](.github/workflows/release.notify.yml)

Send a notification to the specified google chat

##### Secrets
| Name                      | Description                                 | Required |
|---------------------------|---------------------------------------------|----------|
| `google-chat-webhook-url` | The URL for sending messages to Google Chat | true     |

##### Example
```yaml
jobs:
  release:
    name: Release
    uses: cupel-co/workflows/.github/workflows/release.notify.yml@vX.X.X
    with:
      google-chat-webhook-url: "${{ secrets.google-chat-webhook-url }}"
```

### Version
#### Generate
Workflow: [version.generate.yml](.github/workflows/version.generate.yml)

Generate a version from the commit history. 

##### Inputs
| Name        | Description                        | Required | Default       |
|-------------|------------------------------------|----------|---------------|
| `flow-type` | The flow type to use (GITHUB_FLOW) | true     | `GITHUB_FLOW` |

##### Outputs
| Name                           | Description                               |
|--------------------------------|-------------------------------------------|
| `major`                        | The Major value                           |
| `minor`                        | The Minor value                           |
| `patch`                        | The Patch value                           |
| `pre-release-tag`              | The prerelease tag value                  |
| `pre-release-tag-with-dash`    | The prerelease tag with dash value        |
| `pre-release-label`            | The prerelease label value                |
| `pre-release-number`           | The prerelease number value               |
| `weighted-pre-release-number`  | The weighted prerelease number value      |
| `build-meta-data`              | The build metadata value                  |
| `full-build-meta-data`         | The full build metadata value             |
| `major-minor-patch`            | The major minor patch value               |
| `sem-ver`                      | The semantic version value                |
| `assembly-sem-ver`             | The assembly semantic version value       |
| `assembly-sem-file-ver`        | The file assembly semantic version value  |
| `full-sem-ver`                 | The full semantic version value           |
| `informational-version`        | The informational version value           |
| `branch-name`                  | The branch name value                     |
| `escaped-branch-name`          | The escaped branch name value             |
| `sha`                          | The sha value                             |
| `short-sha`                    | The short sha value                       |
| `version-source-sha`           | The version source sha value              |
| `commits-since-version-source` | The commits since version source value    |
| `uncommitted-changes`          | The uncommitted changes value             |
| `commit-date`                  | The commit date value                     |

##### Example
```yaml
jobs:
  version:
    name: Version
    uses: cupel-co/workflows/.github/workflows/version.generate.yml@vX.X.X
```
