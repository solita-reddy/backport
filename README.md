# Backport Action Example

This repository demonstrates the [sorenlouv/backport-github-action](https://github.com/marketplace/actions/backport-action) for automatically backporting merged PRs to release branches.

## How It Works

1. A PR is opened targeting `main` (or any default branch).
2. A reviewer applies a label like `backport-to-release/1.x` to the PR.
3. When the PR is **merged**, the workflow triggers automatically and opens a new backport PR targeting `release/1.x`.
4. If there are merge conflicts, the workflow reports the failure and you can resolve them manually.

## Setup

### 1. GitHub Actions Workflow

The workflow is already defined in [`.github/workflows/backport.yml`](.github/workflows/backport.yml).

It triggers on:
- `pull_request_target` → `labeled` or `closed` events

### 2. Configuration (`.backportrc.json`)

Edit `.backportrc.json` to reflect your repo and target branches:

\`\`\`json
{
  "repoOwner": "YOUR_ORG",
  "repoName": "YOUR_REPO",
  "targetBranchChoices": ["main", "release/1.x", "release/2.x"],
  "branchLabelMapping": {
    "^backport-to-(.+)$": "$1"
  }
}
\`\`\`

### 3. Labels

Create labels in your GitHub repo matching the pattern \`backport-to-<branch>\`. For example:
- \`backport-to-release/1.x\`
- \`backport-to-release/2.x\`

Labels can be created via the GitHub CLI:

\`\`\`sh
gh label create "backport-to-release/1.x" --color 0075ca
gh label create "backport-to-release/2.x" --color e4e669
\`\`\`

## Branch Structure Example

\`\`\`
main          ──●──●──●──●──  (active development)
                        ↓ backport
release/2.x   ──●──────●───   (stable, security/bugfix only)
release/1.x   ──●────────●──  (legacy, critical fixes only)
\`\`\`

## Conflict Handling

If the cherry-pick has conflicts, the action will fail. You can backport manually:

\`\`\`sh
git checkout release/1.x
git cherry-pick <commit-sha>
# resolve conflicts
git cherry-pick --continue
\`\`\`

## Notes

- Uses \`pull_request_target\` (not \`pull_request\`) so the action has the write permissions needed to push backport branches.
- \`GITHUB_TOKEN\` is available automatically — no extra secrets needed.
- Backport PRs are tagged with the \`backport\` label automatically to prevent re-triggering the workflow.
