# Shared Actions Implementation Example

This repository serves as an official reference implementation showcasing how to consume our organization's centralized, cross-platform GitHub Actions workflows. 

It demonstrates how a client repository can trigger multi-job visual pipelines natively in the GitHub UI while keeping core DevOps business logic completely secure and private inside GitLab.

---

## Architectural Workflow

This repository functions as the **Consumer Layer** in our 3-part pipeline architecture:

1. **This Repo (Consumer):** Calls a public structural blueprint workflow and securely hands off local deploy credentials.
2. **Orchestration Hub (`k-int/github-actions`):** Contains the structural reusable skeletons (`workflow_call`) that render the visual dependency graph (DAG) in the GitHub Actions UI.
3. **Core Vault (Private GitLab):** Holds the actual proprietary business logic and shell execution blocks, which are securely cloned at runtime into volatile runner memory.

---

## Repository Structure

Because the pipeline architecture is completely centralized, client applications require zero local scripts, binaries, or composite action folders. The implementation remains incredibly lightweight:

```text
actions-example-repo/
├── README.md                # This documentation
└── .github/
    └── workflows/
        └── hello-test.yml   # The orchestrator caller configuration

```

---

## Reference Implementation

To replicate this pattern in your own application repository, create a workflow file (e.g., `.github/workflows/hello-test.yml`) using the exact structural pattern defined below:

```yaml
name: Hello Test Pipeline

on:
  push:
    branches: [ "main" ]
  pull_request:
  workflow_dispatch: # Allows manual pipeline runs from the Actions tab

jobs:
  core-pipeline:
    # 1. Call the mirrored structural blueprint repository on GitHub
    uses: k-int/github-actions/.github/workflows/hello-world.yml@main
    
    # 2. Explicitly map your repository secrets across the organizational wall
    secrets:
      GITLAB_DEPLOY_USER: ${{ secrets.GITLAB_DEPLOY_USER }}
      GITLAB_DEPLOY_TOKEN: ${{ secrets.GITLAB_DEPLOY_TOKEN }}

```

---

## Required Setup for Downstream Repositories

If you are setting up a new repository to use these workflows, it will fail to authenticate against the core logic vault unless credentials are provided.

### Provisioning Secrets

1. Obtain the read-only deployment token from our password vault (**Keeper Path:** `Libraries/Gitlab DevOps/deploy token READ_ACTIONS_SCRIPTS`).
2. Navigate to your target GitHub repository -> **Settings** -> **Secrets and variables** -> **Actions**.
3. Create the following two **Repository Secrets**:
* `GITLAB_DEPLOY_USER` : *(The username string from Keeper)*
* `GITLAB_DEPLOY_TOKEN` : *(The alphanumeric password/token string from Keeper)*



> 💡 **Pro-Tip:** If you have administrator access to the GitHub Organization, you can define these secrets once at the **Organization Settings** level to automatically share them across all matching repositories safely, removing manual copy-paste overhead.
