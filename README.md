# GitHub Workflows
Reusable workflows for GitHub

## Workflows
### Generate version
Workflow: [generate-version.yml](.github/workflows/generate-version.yml)

Generate a version from the commit history. 

A file named `GitVersion.yml` in the `./` directory is required for this to work, please see [GitVersion.yml](./GitVersion.yml).

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





* [notify-pull-request.yml](.github/workflows/notify-pull-request.yml)
* [opentofu-apply.yml](.github/workflows/opentofu-apply.yml)
* [opentofu-destroy.yml](.github/workflows/opentofu-destroy.yml)
* [opentofu-plan.yml](.github/workflows/opentofu-plan.yml)
* [opentofu-plan-and-apply.yml](.github/workflows/opentofu-plan-and-apply.yml)
* [tag.yml](.github/workflows/tag.yml)
* [tflint.yml](.github/workflows/tflint.yml)
* [tfsec.yml](.github/workflows/tfsec.yml)
