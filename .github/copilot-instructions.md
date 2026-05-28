# Copilot instructions

## Repository purpose

This repository hosts shared **GitHub Actions reusable workflows** for the `cupel-co` organisation. It contains two kinds of workflows under `.github/workflows/`:

- **Reusable workflows** (called via `workflow_call`) — consumed by other repositories. File names use dot-separated topic.action form (e.g. `pull-request.notify.yml`, `release.create.yml`, `version.generate.yml`).
- **Repository workflows** — automation for this repository itself (e.g. `integrate.yml`, `preview.yml`). File names use dash-separated form (e.g. `pull-request-notify.yml`).

## Conventions

- **YAML style**: 2-space indent, single quotes for strings that contain `${{ }}` expressions, lowercase kebab-case for input, output, and secret names.
- **Naming**:
    - Reusable workflow files: `<topic>.<action>.yml` (e.g. `release.create.yml`).
    - Wrapper workflow files in this repo: `<topic>-<action>.yml`.
    - Job ids: short, lowercase (e.g. `notify`, `release`, `version`). Job `name:` is Title Case.
- **Actions pinning**: always pin third-party and `cupel-co/actions` references to a **full commit SHA**, not a tag or branch.
- **Concurrency**: long-running or release-producing workflows set a `concurrency` group keyed on `${{ github.workflow }}-${{ github.ref_name }}` with `cancel-in-progress: false`.
- **Permissions**: declare the minimum required `permissions:` block at the job level in examples and wrappers. Do not grant broader scopes than needed.
- **Secrets**: declare every secret explicitly under `workflow_call.secrets` with a `description` and `required:` field. Pass through using lowercase secret names (e.g. `google-chat-webhook-url`), mapped from uppercase repository secrets (e.g. `GOOGLE_CHAT_WEBHOOK_URL`).
- **Inputs/Outputs**: declare every input/output with a `description`. Inputs include `required` and (where applicable) `default`.

## When adding or editing a reusable workflow

1. Place it in `.github/workflows/` using `<topic>.<action>.yml`.
2. Use `on: workflow_call:` with fully described `inputs`, `outputs`, and `secrets`.
3. Pin any `uses:` reference to a commit SHA.
4. Update `README.md`:
    - Add a section under **Reusable Workflows** matching existing structure: workflow link, short description, then sub-sections for **Inputs**, **Outputs**, **Secrets**, **Required Permissions**, and an **Example** YAML block.
    - Use `@vX.X.X` as the version placeholder in examples.
    - Keep ordering consistent with existing entries.
5. If adding a wrapper for this repo, also create the dash-named file in `.github/workflows/` and document it under **Repository Workflows** in `README.md`.

## When editing repository (wrapper) workflows

- Reference local reusable workflows with `uses: ./.github/workflows/<name>.yml` (not by ref).
- Keep triggers minimal and explicit (`pull_request` types, `push` branches, etc.).
- Mirror secret/permission requirements of the reusable workflow being called.

## Release & versioning

- Versioning is driven by `version.generate.yml` (GitVersion, `GITHUB_FLOW`). Do not hand-edit version numbers.
- Releases are produced by `integrate.yml` on push to `main`, tagging as `v${{ sem-ver }}`.
- Breaking changes to a reusable workflow's inputs/outputs/secrets must be called out in the PR description; consumers pin by tag.

## Documentation

- `README.md` is the source of truth for workflow contracts — keep it in sync with every workflow change in the same PR.
- Tables use the existing column layout; align pipes for readability.
