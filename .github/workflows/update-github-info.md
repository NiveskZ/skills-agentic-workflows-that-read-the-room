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
    - awesome-copilot.github.com
tools:
  web-fetch: {}
  edit: {}
sandbox:
  agent: 
    config:
      filesystem: {}
safe-outputs:
  create-pull-request:
    title-prefix: "[mona] "
    draft: true
    fallback-as-issue: false 
env:
  PR_TITLE: "Mona GitHub Info website update"
  PR_BRANCH_PREFIX: "auto/update-github-info"
---

# Update Mona's GitHub Info website

Read `notes/mona-notes.md` before making changes.

Use these sources:
- `notes/mona-notes.md`
- GitHub Blog: https://github.blog/latest/
- GitHub Changelog: https://github.blog/changelog/
- awesome-copilot workflows: https://awesome-copilot.github.com/workflows/

Use the `web-fetch` tool to fetch https://awesome-copilot.github.com/workflows/ and use it as an additional source for Mona's GitHub Info updates.
Update exactly `site/content/github-info.md` with concise, practical updates for readers and include source context when content comes from the GitHub Blog, GitHub Changelog, or awesome-copilot workflows.
Change only `site/content/github-info.md` in the generated pull request; do not modify other repository files.

Open a draft pull request for Mona to review that updates `site/content/github-info.md`.
Use a pull request title that mentions Mona, GitHub Info, or a website update.
Do not write directly to `main`; rely on `safe-outputs` with `create-pull-request`.

## Important: Patch Format Requirements

When using the `edit` tool to modify files, generate patches in this exact format:

```
*** Begin Patch
--- a/path/to/file
+++ b/path/to/file
@@ -line,count +line,count @@
 context line
-removed line
+added line
 context line
*** End Patch
```

The patch MUST start with `*** Begin Patch` and end with `*** End Patch`. Include proper `---` and `+++` file headers with relative paths, and use standard unified diff format with `@@` hunk headers.
