---
name: Update GitHub Info
on:
  schedule:
    - cron: "0 6 * * *" # daily at 06:00 UTC
  workflow_dispatch: {}
permissions:
  contents: read
  pull-requests: read
network:
  allowed:
    - github.blog
    - github.com
tools:
  web-fetch: {}
sandbox:
  agent: 
    config:
      filesystem: {}
safe-outputs:
  create-pull-request:
    title-prefix: "[mona]"
    draft: true
    fallback-as-issue: false 
env:
  PR_TITLE: "Update GitHub info (automated)"
  PR_BRANCH_PREFIX: "auto/update-github-info"
---

Update `site/content/github-info.md` from the latest GitHub blog updates and open a pull request for Mona to review.

## Usage

Runs daily (fuzzy) and on demand. Uses allowed tools to fetch web pages, read local notes, and propose changes via safe outputs.

## Steps (agent)

1. Read `notes/mona-notes.md` for context and phrasing.
2. Web fetch and summarize:
   - https://github.blog/latest/
   - https://github.blog/changelog/
3. Update `site/content/github-info.md` with a short "Latest GitHub Updates" section (2–6 bullets), annotate entries with the date and source URL.
4. Create a branch `${{ env.PR_BRANCH_PREFIX }}/YYYYMMDD`, commit the change, and push.
5. Emit the `create-pull-request` safe output with branch, title, body summarizing changes and fetched sources; request Mona's review.

## Constraints

- Do not write directly to `main` — use the safe output to propose a pull request.
- Use only the declared `tools`.
- If web fetch fails, include an explanatory note in the PR body and only use local notes to update content.

## Outputs

- `create-pull-request` safe output containing branch name, PR title, PR body, and changed files.
