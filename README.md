# GitHub Workflows
[![Integrate](https://github.com/cupel-co/workflows/actions/workflows/integrate.yml/badge.svg?branch=main)](https://github.com/cupel-co/workflows/actions/workflows/integrate.yml?query=branch%3Amain)

Shared GitHub Actions workflows for Cupel repositories.

## Overview

This repository contains two kinds of workflows:

- **Reusable workflows** that can be called from other repositories with `workflow_call`
- **Repository workflows** that wire the reusable workflows together for this repository

## Reusable Workflows

### Pull Request

#### Notify

Workflow: [`pull-request.notify.yml`](.github/workflows/pull-request.notify.yml)

Send a Google Chat notification for a pull request event.

##### Secrets

| Name                      | Description                                          | Required |
|---------------------------|------------------------------------------------------|----------|
| `google-chat-webhook-url` | The webhook URL used to send messages to Google Chat | true     |

##### Example

```yaml
jobs:
  pull-request:
    name: Pull Request
    uses: cupel-co/workflows/.github/workflows/pull-request.notify.yml@vX.X.X
    secrets:
      google-chat-webhook-url: ${{ secrets.GOOGLE_CHAT_WEBHOOK_URL }}
```

#### Update

Workflow: [`pull-request.update.yml`](.github/workflows/pull-request.update.yml)

Update the pull request description using repository and issue metadata.

##### Supported Placeholders

| Name                      | Description                                   |
|---------------------------|-----------------------------------------------|
| `{{BRANCH_NAME}}`         | The branch name                               |
| `{{ENVIRONMENT}}`         | The environment name                          |
| `{{ISSUE_NUMBER}}`        | The issue number                              |
| `{{ISSUE_TITLE}}`         | The issue title                               |
| `{{PULL_REQUEST_NUMBER}}` | The pull request number                       |
| `{{PULL_REQUEST_TITLE}}`  | The pull request title                        |
| `{{REPOSITORY}}`          | The full repository name, including the owner |

##### Inputs

| Name           | Description                                              | Required | Default |
|----------------|----------------------------------------------------------|----------|---------|
| `environment`  | The environment name to inject into the PR description   | true     |         |

##### Required Permissions

| Permission            | Access  |
|-----------------------|---------|
| `issues`              | `read`  |
| `pull-requests`       | `write` |
| `repository-projects` | `read`  |

##### Outputs

| Name          | Description                           |
|---------------|---------------------------------------|
| `description` | The rendered pull request description |

##### Example

```yaml
jobs:
  pull-request:
    name: Pull Request
    uses: cupel-co/workflows/.github/workflows/pull-request.update.yml@vX.X.X
    permissions:
      issues: read
      pull-requests: write
      repository-projects: read
    with:
      environment: Preview
```

### Release

#### Create

Workflow: [`release.create.yml`](.github/workflows/release.create.yml)

Create a signed tag, publish a GitHub release, and send a Google Chat notification.

##### Inputs

| Name   | Description                        | Required | Default |
|--------|------------------------------------|----------|---------|
| `tag`  | The Git tag to create and release  | true     |         |

##### Secrets

| Name                       | Description                                                       | Required |
|----------------------------|-------------------------------------------------------------------|----------|
| `google-chat-webhook-url`  | The webhook URL used to send release notifications to Google Chat | true     |
| `gpg-private-key`          | The armored GPG private key used to sign the Git tag              | true     |
| `gpg-private-key-password` | The password for the GPG private key                              | true     |
| `token`                    | A GitHub token with permission to push tags and create releases   | true     |

##### Required Permissions

| Permission | Access  |
|------------|---------|
| `contents` | `write` |

##### Example

```yaml
jobs:
  release:
    name: Release
    uses: cupel-co/workflows/.github/workflows/release.create.yml@vX.X.X
    permissions:
      contents: write
    with:
      tag: v0.0.1
    secrets:
      google-chat-webhook-url: ${{ secrets.GOOGLE_CHAT_WEBHOOK_URL }}
      gpg-private-key: ${{ secrets.GH_GPG_PRIVATE_KEY }}
      gpg-private-key-password: ${{ secrets.GH_GPG_PRIVATE_KEY_PASSWORD }}
      token: ${{ secrets.GH_RELEASE_PAT }}
```

#### Notify

Workflow: [`release.notify.yml`](.github/workflows/release.notify.yml)

Send a release notification to Google Chat.

##### Inputs

| Name  | Description                                     | Required | Default |
|-------|-------------------------------------------------|----------|---------|
| `tag` | The release tag to include in the notification  | true     |         |

##### Secrets

| Name                       | Description                                           | Required |
|----------------------------|-------------------------------------------------------|----------|
| `google-chat-webhook-url`  | The webhook URL used to send messages to Google Chat  | true     |

##### Example

```yaml
jobs:
  release:
    name: Release
    uses: cupel-co/workflows/.github/workflows/release.notify.yml@vX.X.X
    with:
      tag: v0.0.1
    secrets:
      google-chat-webhook-url: ${{ secrets.GOOGLE_CHAT_WEBHOOK_URL }}
```

### Version

#### Generate

Workflow: [`version.generate.yml`](.github/workflows/version.generate.yml)

Generate semantic version information from the repository history.

##### Inputs

| Name        | Description                     | Required | Default       |
|-------------|---------------------------------|----------|---------------|
| `flow-type` | The versioning flow type to use | false    | `GITHUB_FLOW` |

##### Outputs

| Name                           | Description                                          |
|--------------------------------|------------------------------------------------------|
| `major`                        | The major version                                    |
| `minor`                        | The minor version                                    |
| `patch`                        | The patch version                                    |
| `pre-release-tag`              | The prerelease tag                                   |
| `pre-release-tag-with-dash`    | The prerelease tag prefixed with a dash when present |
| `pre-release-label`            | The prerelease label                                 |
| `pre-release-number`           | The prerelease number                                |
| `weighted-pre-release-number`  | The weighted prerelease number                       |
| `build-meta-data`              | The build metadata                                   |
| `full-build-meta-data`         | The full build metadata                              |
| `major-minor-patch`            | The major.minor.patch value                          |
| `sem-ver`                      | The semantic version                                 |
| `assembly-sem-ver`             | The assembly semantic version                        |
| `assembly-sem-file-ver`        | The assembly file version                            |
| `full-sem-ver`                 | The full semantic version                            |
| `informational-version`        | The informational version                            |
| `branch-name`                  | The branch name                                      |
| `escaped-branch-name`          | The escaped branch name                              |
| `sha`                          | The full commit SHA                                  |
| `short-sha`                    | The short commit SHA                                 |
| `version-source-sha`           | The version source SHA                               |
| `commits-since-version-source` | The number of commits since the version source       |
| `uncommitted-changes`          | Whether uncommitted changes were detected            |
| `commit-date`                  | The commit date                                      |

##### Example

```yaml
jobs:
  version:
    name: Version
    uses: cupel-co/workflows/.github/workflows/version.generate.yml@vX.X.X
```

## Repository Workflows

The following workflows are defined in this repository for its own automation and are not intended to be consumed from other repositories directly.

### [`integrate.yml`](.github/workflows/integrate.yml)

Runs on pushes to `main`, generates a version, creates a release, and sends a notification.

### [`preview.yml`](.github/workflows/preview.yml)

Runs on pull request open, reopen, and synchronize events to generate version information for preview builds.

### [`pull-request-notify.yml`](.github/workflows/pull-request-notify.yml)

Runs on pull request open and reopen events and delegates to the reusable `pull-request.notify.yml` workflow.

### [`pull-request-update.yml`](.github/workflows/pull-request-update.yml)

Runs on pull request open and reopen events and delegates to the reusable `pull-request.update.yml` workflow with the `Preview` environment.
