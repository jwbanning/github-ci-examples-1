# GitHub CI Examples

This repository contains examples and utilities for GitHub CI/CD integration with Postman.

## Contents

- `.github/workflows` - The pipelines
- `postman/schemas/` - API specification schemas (JSON and YAML formats)


## Examples

### Bidirectional Spec Synch (GitHub <-> Postman)

Syncs a spec in Postman with the contents of a file in the repo, or vice-versa.  For detailed info and examples see [POSTMAN-SYNC-EXAMPLES.md](https://github.com/postman-solutions-eng/github-ci-examples/blob/main/.github/POSTMAN_SYNC_EXAMPLES.md).: 



## Getting Started

#### 1. Create Postman API Key

1. Go to [Postman Account Settings](https://go.postman.co/settings/me/api-keys)
2. Click **Generate API Key**
3. Name it (e.g., "GitHub Actions Sync")
4. Copy the API key

#### 2. Clone this repo and push it your own GitHub org

#### 3. Add API Key to GitHub Secrets

1. Go to your GitHub repository
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Name: `POSTMAN_API_KEY`
5. Value: Paste your Postman API key
6. Click **Add secret**


