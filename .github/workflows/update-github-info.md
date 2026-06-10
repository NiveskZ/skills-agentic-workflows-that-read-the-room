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

Follow these steps in order:

1. Read `notes/mona-notes.md` for editorial guidelines.
2. Read `site/content/github-info.md` to understand the current content.
3. Use the `web-fetch` tool to fetch `https://github.blog/latest/` for the latest GitHub Blog posts.
4. Use the `web-fetch` tool to fetch `https://github.blog/changelog/` for the latest GitHub Changelog entries.
5. Use the `web-fetch` tool to fetch `https://awesome-copilot.github.com/workflows/` for awesome-copilot workflow examples.
6. Update `site/content/github-info.md` with concise, practical updates based on what you fetched. Include source context when content comes from the GitHub Blog, GitHub Changelog, or awesome-copilot workflows.
7. Change only `site/content/github-info.md`; do not modify any other repository files.
8. Open a draft pull request for Mona to review with a title that mentions Mona, GitHub Info, or a website update. Do not write directly to `main`; rely on `safe-outputs` with `create-pull-request`.

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