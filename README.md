# GitHub Workflows
[![Integrate](https://github.com/cupel-co/workflows/actions/workflows/integrate.yml/badge.svg?branch=main)](https://github.com/cupel-co/workflows/actions/workflows/integrate.yml?query=branch%3Amain)

Reusable workflows for GitHub

## Workflows
### OpenTofu
#### Apply
Workflow: [opentofu.apply.yml](.github/workflows/opentofu.apply.yml)

Apply OpenTofu changes.

##### Inputs
| Name                  | Description                                                                                   | Required | Default |
|-----------------------|-----------------------------------------------------------------------------------------------|----------|---------|
| `artifact-name`       | The name of the artifact to download for the apply step                                       | true     |         |
| `environment`         | The target environment where the apply operation will be executed (e.g., staging, production) | true     |         |
| `opentofu-apply-args` | Additional arguments to pass to the tofu apply command                                        | false    |         |
| `oidc-role-arn`       | The ARN of the OIDC role to assume for AWS credentials                                        | true     |         |
| `region`              | The AWS region to use for the apply operation                                                 | true     |         |
| `opentofu-version`    | The version of OpenTofu to install                                                            | false    | `1.8.1` |

##### Secrets
| Name                  | Description                                                                                 | Required |
|-----------------------|---------------------------------------------------------------------------------------------|----------|
| `opentofu-apply-args` | Additional arguments that contain secret values that need to be passed to the apply command | false    |

##### Example
```yaml
jobs:
  apply:
    name: Apply
    uses: cupel-co/workflows/.github/workflows/opentofu.apply@vX.X.X
    with:
      artifact-name: cicd-production
      environment: Production
      opentofu-apply-args: '-var-file="variables/production.tfvars"'
      oidc-role-arn: arn:aws:iam::012345678901:role/GitHub
      region: ap-southeast-2
    secrets:
      opentofu-apply-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
```

#### Cost
Workflow: [opentofu.cost.yml](.github/workflows/opentofu.cost.yml)

Add a comment to a PR for Infracost changes. The underlying actions include a default template to use. The name of the workspace is derived from the environment variables filename.

##### Inputs
| Name                           | Description                                                             | Required | Default            |
|--------------------------------|-------------------------------------------------------------------------|----------|--------------------|
| `base-ref`                     | The name of the base ref, generally this is main, to checkout           | false    | `main`             |
| `infracost-comment-behaviour`  | The behaviour for the comment                                           | false    | `update`           |
| `infracost-comment-identifier` | A unique identifier to for the comment added by this implementation     | false    | `comment`          |
| `infracost-currency`           | The currency to show estimates in                                       | false    | `AUD`              |
| `infracost-template`           | The infracost template                                                  | false    |                    |
| `infracost-usage`              | The usage file contents                                                 | false    |                    |
| `infracost-version`            | The version of Infracost to install                                     | false    | `0.10.x`           |
| `infracost-workspace-prefix`   | The prefix for the workspace name                                       | false    |                    |
| `head-ref`                     | The name of the head ref, generally this is feature branch, to checkout | true     |                    |
| `opentofu-version`             | The version of Opentofu to install                                      | false    | `1.8.1`            |
| `pull-request-number`          | The PR number                                                           | true     |                    |
| `repository`                   | The repository name                                                     | true     |                    |
| `working-directory`            | The directory where the infrastructure code is located.                 | false    | `./infrastructure` |

##### Secrets
| Name                | Description           | Required |
|---------------------|-----------------------|----------|
| `infracost-api-key` | The infracost API key | true     |
| `token`             | The GitHub token      | true     |

##### Example
###### Using template
```yaml
jobs:
  cost:
    name: Cost
    uses: cupel-co/workflows/.github/workflows/opentofu.cost.yml@vX.X.X
    permissions:
      contents: read
      pull-requests: write
    with:
      base-ref: main
      head-ref: ${{ github.ref_name }}
      infracost-comment-behaviour: update
      infracost-currency: AUD
      infracost-template: |
        version: 0.1
        projects:
        {{- range `$project := matchPaths "infrastructure/variables/:env.tfvars" }}
            - path: "./infrastructure"
              name: "{{ `$project.env }}"
              terraform_workspace: "${{ inputs.workspace-prefix }}{{ `$project.env }}"
              terraform_var_files:
                - "{{ relPath "./infrastructure" `$project._path }}"
              dependency_paths:
                - "**"
                - "{{ relPath "./infrastructure" `$project._path }}"
        {{- end }}
      infracost-usage:

      infracost-version: 0.10.x
    secrets:
      infracost-api-key: ${{ secrets.INFRACOST_API_KEY }}
```
###### Using workspace-prefix
```yaml
jobs:
  cost:
    name: Cost
    uses: cupel-co/workflows/.github/workflows/opentofu.cost.yml@vX.X.X
    permissions:
      contents: read
      pull-requests: write
    with:
      base-ref: main
      head-ref: ${{ github.ref_name }}
      infracost-comment-behaviour: update
      infracost-currency: AUD
      infracost-workspace-prefix: github-repositories-
      infracost-usage:

      infracost-version: 0.10.x
    secrets:
      infracost-api-key: ${{ secrets.INFRACOST_API_KEY }}
```

#### Destroy
Workflow: [opentofu.destroy.yml](.github/workflows/opentofu.destroy.yml)

Destroy resources.

##### Inputs
| Name                           | Description                                                        | Required | Default            |
|--------------------------------|--------------------------------------------------------------------|----------|--------------------|
| `oidc-role-arn`                | The ARN of the OIDC role to assume for AWS credentials             | true     |                    |
| `opentofu-destroy-args`        | Additional arguments to pass to the tofu destroy command           | false    |                    |
| `opentofu-init-args`           | Additional arguments to pass to the tofu init command              | false    |                    |
| `opentofu-use-primary-backend` | Specifies whether to use the primary backend during initialization | false    | `true`             |
| `opentofu-version`             | The version of OpenTofu to install                                 | false    | `1.8.1`            |
| `opentofu-workspace`           | The workspace to select or create during the apply process         | true     |                    |
| `region`                       | The AWS region to use for the apply operation                      | true     |                    |
| `working-directory`            | The directory where the infrastructure code is located             | false    | `./infrastructure` |

##### Secrets
| Name                    | Description                                              | Required |
|-------------------------|----------------------------------------------------------|----------|
| `opentofu-destroy-args` | Additional arguments to pass to the tofu destroy command | false    |
| `opentofu-init-args`    | Additional arguments to pass to the tofu init command    | false    |

##### Example
```yaml
jobs:
  apply:
    name: Apply
    uses: cupel-co/workflows/.github/workflows/opentofu.destroy@vX.X.X
    with:
      oidc-role-arn: arn:aws:iam::012345678901:role/GitHub
      opentofu-destroy-args: '-var-file="variables/production.tfvars"'
      opentofu-init-args: '-var-file="variables/production.tfvars"'
      opentofu-use-primary-backend: false
      opentofu-workspace: cicd-production
      region: ap-southeast-2
    secrets:
      opentofu-apply-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
      opentofu-init-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
```

#### Lint
Workflow: [opentofu.lint.yml](.github/workflows/opentofu.lint.yml)

Lint OpenTofu files

##### Inputs
| Name                  | Description                                            | Required | Default            |
|-----------------------|--------------------------------------------------------|----------|--------------------|
| `working-directory`   | The directory where the infrastructure code is located | false    | `./infrastructure` |

##### Example
```yaml
jobs:
  scan:
    name: Scan
    uses: cupel-co/workflows/.github/workflows/opentofu.lint.yml@vX.X.X
```

#### Plan
Workflow: [opentofu.plan.yml](.github/workflows/opentofu.plan.yml)

Generate an OpenTofu plan for specified resources.

##### Inputs
| Name                           | Description                                                        | Required | Default            |
|--------------------------------|--------------------------------------------------------------------|----------|--------------------|
| `artifact-name`                | The name of the artifact to upload after the plan step             | false    |                    |
| `oidc-role-arn`                | The ARN of the OIDC role to assume for AWS credentials             | true     |                    |
| `opentofu-init-args`           | Additional arguments to pass to the tofu init command              | false    |                    |
| `opentofu-plan-args`           | Additional arguments to pass to the tofu plan command              | false    |                    |
| `opentofu-use-primary-backend` | Specifies whether to use the primary backend during initialization | false    | `true`             |
| `opentofu-version`             | The version of OpenTofu to install                                 | false    | `1.8.1`            |
| `opentofu-workspace`           | The workspace to select or create during the apply process         | true     |                    |
| `region`                       | The AWS region to use for the apply operation                      | true     |                    |
| `working-directory`            | The directory where the infrastructure code is located             | false    | `./infrastructure` |

##### Secrets
| Name                 | Description                                           | Required |
|----------------------|-------------------------------------------------------|----------|
| `opentofu-init-args` | Additional arguments to pass to the tofu init command | false    |
| `opentofu-plan-args` | Additional arguments to pass to the tofu plan command | false    |

##### Example
```yaml
jobs:
  plan:
    name: Plan
    uses: cupel-co/workflows/.github/workflows/opentofu.plan@vX.X.X
    with:
      artifact-name: cicd-production
      oidc-role-arn: arn:aws:iam::012345678901:role/GitHub
      opentofu-init-args: '-var-file="variables/production.tfvars"'
      opentofu-plan-args: '-var-file="variables/production.tfvars"'
      opentofu-use-primary-backend: false
      opentofu-workspace: cicd-production
      region: ap-southeast-2
    secrets:
      opentofu-init-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
      opentofu-plan-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
```

#### Plan and Apply
Workflow: [opentofu.plan-and-apply.yml](.github/workflows/opentofu.plan-and-apply.yml)

Plan and apply OpenTofu changes

##### Inputs
| Name                           | Description                                                                                   | Required | Default            |
|--------------------------------|-----------------------------------------------------------------------------------------------|----------|--------------------|
| `artifact-name`                | The name of the artifact to upload after the plan step                                        | true     |                    |
| `environment`                  | The target environment where the apply operation will be executed (e.g., staging, production) | true     |                    |
| `oidc-role-arn`                | The ARN of the OIDC role to assume for AWS credentials                                        | true     |                    |
| `opentofu-apply-args`          | Additional arguments to pass to the tofu apply command                                        | false    |                    |
| `opentofu-init-args`           | Additional arguments to pass to the tofu init command                                         | false    |                    |
| `opentofu-plan-args`           | Additional arguments to pass to the tofu plan command                                         | false    |                    |
| `opentofu-use-primary-backend` | Specifies whether to use the primary backend during initialization                            | false    | `true`             |
| `opentofu-version`             | The version of OpenTofu to install                                                            | false    | `1.8.1`            |
| `opentofu-workspace`           | The workspace to select or create during the apply process                                    | true     |                    |
| `region`                       | The AWS region to use for the apply operation                                                 | true     |                    |
| `working-directory`            | The directory where the infrastructure code is located                                        | false    | `./infrastructure` |

##### Secrets
| Name                      | Description                                           | Required |
|---------------------------|-------------------------------------------------------|----------|
| `opentofu-apply-args`     | Additional arguments to pass to the tofu init command | false    |
| `google-chat-webhook-url` | The Webhook URL for the chat to send the message to   | false    |
| `opentofu-init-args`      | Additional arguments to pass to the tofu init command | false    |
| `opentofu-plan-args`      | Additional arguments to pass to the tofu plan command | false    |

##### Example
```yaml
jobs:
  deploy:
    name: Deploy
    uses: cupel-co/workflows/.github/workflows/opentofu.plan-and-apply@vX.X.X
    with:
      artifact-name: cicd-production
      environment: production
      oidc-role-arn: arn:aws:iam::012345678901:role/GitHub
      opentofu-init-args: '-var-file="variables/production.tfvars"'
      opentofu-plan-args: '-var-file="variables/production.tfvars"'
      opentofu-use-primary-backend: false
      opentofu-workspace: cicd-production
      region: ap-southeast-2
    secrets:
      google-chat-webhook-url: ${{ secrets.GOOGLE_CHAT_WEBHOOK_URL }}
      opentofu-init-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
      opentofu-plan-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
```

#### Sec
Workflow: [opentofu.sec.yml](.github/workflows/opentofu.sec.yml)

Analyse OpenTofu code

##### Inputs
| Name                  | Description                                            | Required | Default            |
|-----------------------|--------------------------------------------------------|----------|--------------------|
| `working-directory`   | The directory where the infrastructure code is located | false    | `./infrastructure` |

##### Example
```yaml
jobs:
  scan:
    name: Scan
    uses: cupel-co/workflows/.github/workflows/opentofu.sec.yml@vX.X.X
```

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
| Name           | Description                                      | Required |
|----------------|--------------------------------------------------|----------|
| `gpg-key`      | The private GPG key used for signing the Git tag | true     |
| `gpg-password` | The passphrase for the GPG private key           | true     |
| `token`        | Token for github interaction                     | true     |

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
      gpg-key: ${{ secrets.GH_GPG_PRIVATE_KEY }}
      gpg-password: ${{ secrets.GH_GPG_PRIVATE_KEY_PASSWORD }}
      token: ${{ github.token }}
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

