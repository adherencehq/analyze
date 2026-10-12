# Adherence analysis

The GitHub Actions workflow that connects a repository to [Adherence](https://adherence.example),
which checks whether your code implements your compliance policies.

> **Status: not yet released.** There is no `v1` tag yet. These steps describe the release.

## What runs, and what leaves your CI

This workflow, at the tag you call, runs a pinned version of the open-source
[`adherence-analysis`](https://pypi.org/project/adherence-analysis/). Adherence rejects runs from
any other workflow.

**Policies** are uploaded to Adherence, not kept in your repository. Adherence extracts their
requirements and sends each one to CI: its rule as structured fields, and its document's name.

**Code** stays in your repository. The workflow reads source files only and splits them into
symbols: classes, methods, constants and declarations. Supported languages: Ruby.

| Sent from CI to Adherence | Stored by Adherence |
|---|---|
| Each symbol's name, file path, line numbers and fingerprint: a SHA-256 hash of its syntax, ignoring comments and whitespace, so the hash changes only when the logic changes | Only for symbols linked as evidence |
| The code of the symbols ranked most likely to implement each requirement | **Never.** Held in memory for the request |
| Scope: files analyzed, languages, excluded paths, files that failed to parse | Yes |

Only default-branch commits produce evidence. Other commits are previews.

## Setup

You need a repository admin for step 1.

### 1. Install the Adherence GitHub App

Install it on the repositories Adherence should analyze. It asks for:

| Permission | Why |
|---|---|
| Contents: read | Shows reviewers the code behind each link, and recomputes hashes at attestation and export |
| Actions: write | Starts this workflow when your compliance officer requests a new evaluation |
| Metadata: read | Required by GitHub for every app |

### 2. Add the workflow file

Add `.github/workflows/adherence.yml` on your default branch:

```yaml
name: Adherence

on:
  push:
    branches: [main] # your default branch
  workflow_dispatch:
    inputs:
      run_id:
        description: Set by Adherence when it starts a run; leave empty when running by hand.
        required: false
        default: ""

permissions:
  contents: read
  id-token: write # the GitHub OIDC token identifies this run to Adherence; no secret is stored

jobs:
  analyze:
    uses: adherencehq/analyze/.github/workflows/analyze.yml@v1
    with:
      run_id: ${{ inputs.run_id }}
```

You store no secrets. Do not add a `pull_request` trigger: runs on unmerged code are not
evidence.

### 3. Run it once

In your repository's **Actions** tab, choose **Adherence**, then **Run workflow**. When it
finishes, the run appears in Adherence and its proposed links are ready for review.

## Versions

`@v1` follows the latest compatible release. To pin an exact release, use a full tag such as
`@v1.0.0`. Adherence accepts supported versions of this workflow only.

## License

[Apache License 2.0](LICENSE)
