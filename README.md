# GitHub Workflows
[![Deploy](https://github.com/cupel-co/workflows/actions/workflows/deploy.yml/badge.svg?branch=main)](https://github.com/cupel-co/workflows/actions/workflows/deploy.yml?query=branch%3Amain)

Reusable workflows for GitHub

## Workflows
### Generate version
Workflow: [generate-version.yml](.github/workflows/generate-version.yml)

Generate a version from the commit history. 

A file named `GitVersion.yml` in the `./` directory is required for this to work, please see [GitVersion.yml](./GitVersion.yml).

#### Inputs
| Name        | Description                        | Required | Default       |
|-------------|------------------------------------|----------|---------------|
| `flow-type` | The flow type to use (GITHUB_FLOW) | true     | `GITHUB_FLOW` |

#### Outputs
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

#### Example
```yaml
jobs:
  generate:
    name: Generate
    uses: cupel-co/workflows/.github/workflows/generate-version.yml@vX.X.X
```

### GitHub Release
Workflow: [github-release.yml](.github/workflows/github-release.yml)

Tag, create a GitHub release and notify.

#### Inputs
| Name   | Description   | Required | Default |
|--------|---------------|----------|---------|
| `tags` | The tag value | true     |         |

#### Secrets
| Name           | Description                                      | Required | Default |
|----------------|--------------------------------------------------|----------|---------|
| `gpg-key`      | The private GPG key used for signing the Git tag | true     |         |
| `gpg-password` | The passphrase for the GPG private key           | true     |         |
| `token`        | Token for github interaction                     | true     |         |

#### Example
```yaml
jobs:
  release:
    name: Release
    uses: ./.github/workflows/release.yml
    needs:
      - generate
    permissions:
      contents: write
    with:
      tag: "v0.0.1"
    secrets:
      gpg-key: ${{ secrets.GH_GPG_PRIVATE_KEY }}
      gpg-password: ${{ secrets.GH_GPG_PRIVATE_KEY_PASSWORD }}
      token: ${{ github.token }}
```

### Notify Pull Request
Workflow: [notify-pull-request.yml](.github/workflows/notify-pull-request.yml)

Send a notification to the specified google chat

#### Secrets
| Name                      | Description                                 | Required | Default |
|---------------------------|---------------------------------------------|----------|---------|
| `google-chat-webhook-url` | The URL for sending messages to Google Chat | true     |         |

#### Example
```yaml
jobs:
  generate:
    name: Generate
    uses: cupel-co/workflows/.github/workflows/notify-pull-request.yml@vX.X.X
    with:
      google-chat-webhook-url: "${{ secrets.google-chat-webhook-url }}"
```

### Notify Release
Workflow: [notify-release.yml](.github/workflows/notify-release.yml)

Send a notification to the specified google chat

#### Secrets
| Name                      | Description                                 | Required | Default |
|---------------------------|---------------------------------------------|----------|---------|
| `google-chat-webhook-url` | The URL for sending messages to Google Chat | true     |         |

#### Example
```yaml
jobs:
  generate:
    name: Generate
    uses: cupel-co/workflows/.github/workflows/notify-release.yml@vX.X.X
    with:
      google-chat-webhook-url: "${{ secrets.google-chat-webhook-url }}"
```

### OpenTofu Apply
Workflow: [opentofu-apply.yml](.github/workflows/opentofu-apply.yml)

Apply OpenTofu changes.

#### Inputs
| Name            | Description                                                                                   | Required | Default |
|-----------------|-----------------------------------------------------------------------------------------------|----------|---------|
| `artifact-name` | The name of the artifact to download for the apply step                                       | true     |         |
| `environment`   | The target environment where the apply operation will be executed (e.g., staging, production) | true     |         |
| `apply-args`    | Additional arguments to pass to the tofu apply command                                        | false    |         |
| `oidc-role-arn` | The ARN of the OIDC role to assume for AWS credentials                                        | true     |         |
| `region`        | The AWS region to use for the apply operation                                                 | true     |         |
| `version`       | The version of OpenTofu to install                                                            | false    | `1.8.1` |

#### Secrets
| Name         | Description                                                                                 | Required | Default |
|--------------|---------------------------------------------------------------------------------------------|----------|---------|
| `apply-args` | Additional arguments that contain secret values that need to be passed to the apply command | false    |         |

#### Example
```yaml
jobs:
  apply:
    name: Apply
    uses: cupel-co/workflows/.github/workflows/opentofu-apply@vX.X.X
    with:
      artifact-name: cicd-production
      environment: Production
      apply-args: '-var-file="variables/production.tfvars"'
      oidc-role-arn: arn:aws:iam::012345678901:role/GitHub
      region: ap-southeast-2
    secrets:
      apply-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
```

### OpenTofu Destroy
Workflow: [opentofu-destroy.yml](.github/workflows/opentofu-destroy.yml)

Destroy resources.

#### Inputs
| Name                  | Description                                                        | Required | Default            |
|-----------------------|--------------------------------------------------------------------|----------|--------------------|
| `destroy-args`        | Additional arguments to pass to the tofu destroy command           | false    |                    |
| `init-args`           | Additional arguments to pass to the tofu init command              | false    |                    |
| `oidc-role-arn`       | The ARN of the OIDC role to assume for AWS credentials             | true     |                    |
| `region`              | The AWS region to use for the apply operation                      | true     |                    |
| `use-primary-backend` | Specifies whether to use the primary backend during initialization | false    | `true`             |
| `version`             | The version of OpenTofu to install                                 | false    | `1.8.1`            |
| `working-directory`   | The directory where the infrastructure code is located             | false    | `./infrastructure` |
| `workspace`           | The workspace to select or create during the apply process         | true     |                    |

#### Secrets
| Name           | Description                                              | Required | Default |
|----------------|----------------------------------------------------------|----------|---------|
| `destroy-args` | Additional arguments to pass to the tofu destroy command | false    |         |
| `init-args`    | Additional arguments to pass to the tofu init command    | false    |         |

#### Example
```yaml
jobs:
  apply:
    name: Apply
    uses: cupel-co/workflows/.github/workflows/opentofu-destroy@vX.X.X
    with:
      destroy-args: '-var-file="variables/production.tfvars"'
      init-args: '-var-file="variables/production.tfvars"'
      oidc-role-arn: arn:aws:iam::012345678901:role/GitHub
      region: ap-southeast-2
      use-primary-backend: false
      workspace: cicd-production
    secrets:
      apply-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
      init-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
```

### OpenTofu Plan
Workflow: [opentofu-plan.yml](.github/workflows/opentofu-plan.yml)

Generate an OpenTofu plan for specified resources.

#### Inputs
| Name                  | Description                                                        | Required | Default            |
|-----------------------|--------------------------------------------------------------------|----------|--------------------|
| `artifact-name`       | The name of the artifact to upload after the plan step             | false    |                    |
| `init-args`           | Additional arguments to pass to the tofu init command              | false    |                    |
| `oidc-role-arn`       | The ARN of the OIDC role to assume for AWS credentials             | true     |                    |
| `plan-args`           | Additional arguments to pass to the tofu plan command              | false    |                    |
| `region`              | The AWS region to use for the apply operation                      | true     |                    |
| `use-primary-backend` | Specifies whether to use the primary backend during initialization | false    | `true`             |
| `version`             | The version of OpenTofu to install                                 | false    | `1.8.1`            |
| `working-directory`   | The directory where the infrastructure code is located             | false    | `./infrastructure` |
| `workspace`           | The workspace to select or create during the apply process         | true     |                    |

#### Secrets
| Name           | Description                                           | Required | Default |
|----------------|-------------------------------------------------------|----------|---------|
| `init-args`    | Additional arguments to pass to the tofu init command | false    |         |
| `plan-args`    | Additional arguments to pass to the tofu plan command | false    |         |

#### Example
```yaml
jobs:
  plan:
    name: Plan
    uses: cupel-co/workflows/.github/workflows/opentofu-plan@vX.X.X
    with:
      artifact-name: cicd-production
      init-args: '-var-file="variables/production.tfvars"'
      oidc-role-arn: arn:aws:iam::012345678901:role/GitHub
      plan-args: '-var-file="variables/production.tfvars"'
      region: ap-southeast-2
      use-primary-backend: false
      workspace: cicd-production
    secrets:
      init-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
      plan-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
```

### OpenTofu Plan and Apply
Workflow: [opentofu-plan-and-apply.yml](.github/workflows/opentofu-plan-and-apply.yml)

Plan and apply OpenTofu changes

#### Inputs
| Name                  | Description                                                                                   | Required | Default            |
|-----------------------|-----------------------------------------------------------------------------------------------|----------|--------------------|
| `apply-args`          | Additional arguments to pass to the tofu apply command                                        | false    |                    |
| `artifact-name`       | The name of the artifact to upload after the plan step                                        | true     |                    |
| `environment`         | The target environment where the apply operation will be executed (e.g., staging, production) | true     |                    |
| `init-args`           | Additional arguments to pass to the tofu init command                                         | false    |                    |
| `oidc-role-arn`       | The ARN of the OIDC role to assume for AWS credentials                                        | true     |                    |
| `plan-args`           | Additional arguments to pass to the tofu plan command                                         | false    |                    |
| `region`              | The AWS region to use for the apply operation                                                 | true     |                    |
| `use-primary-backend` | Specifies whether to use the primary backend during initialization                            | false    | `true`             |
| `version`             | The version of OpenTofu to install                                                            | false    | `1.8.1`            |
| `working-directory`   | The directory where the infrastructure code is located                                        | false    | `./infrastructure` |
| `workspace`           | The workspace to select or create during the apply process                                    | true     |                    |

#### Secrets
| Name           | Description                                                                                 | Required | Default |
|----------------|---------------------------------------------------------------------------------------------|----------|---------|
| `apply-args`   | Additional arguments that contain secret values that need to be passed to the apply command | false    |         |
| `init-args`    | Additional arguments to pass to the tofu init command                                       | false    |         |
| `plan-args`    | Additional arguments to pass to the tofu plan command                                       | false    |         |

#### Example
```yaml
jobs:
  deploy:
    name: Deploy
    uses: cupel-co/workflows/.github/workflows/opentofu-plan-and-apply@vX.X.X
    with:
      artifact-name: cicd-production
      environment: production
      init-args: '-var-file="variables/production.tfvars"'
      oidc-role-arn: arn:aws:iam::012345678901:role/GitHub
      plan-args: '-var-file="variables/production.tfvars"'
      region: ap-southeast-2
      use-primary-backend: false
      workspace: cicd-production
    secrets:
      init-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
      plan-args: '-var="github_pat=${{ secrets.GH_PAT }}"'
```

### TF Lint
Workflow: [tflint.yml](.github/workflows/tflint.yml)

Lint OpenTofu files

#### Inputs
| Name                  | Description                                            | Required | Default            |
|-----------------------|--------------------------------------------------------|----------|--------------------|
| `working-directory`   | The directory where the infrastructure code is located | false    | `./infrastructure` |

#### Example
```yaml
jobs:
  scan:
    name: Scan
    uses: cupel-co/workflows/.github/workflows/tflint@vX.X.X
```

### TF Sec
Workflow: [tfsec.yml](.github/workflows/tfsec.yml)

Analyse OpenTofu code

#### Inputs
| Name                  | Description                                            | Required | Default            |
|-----------------------|--------------------------------------------------------|----------|--------------------|
| `working-directory`   | The directory where the infrastructure code is located | false    | `./infrastructure` |

#### Example
```yaml
jobs:
  scan:
    name: Scan
    uses: cupel-co/workflows/.github/workflows/tfsec@vX.X.X
```
