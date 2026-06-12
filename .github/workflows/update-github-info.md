---
name: Update GitHub Info
on:
  workflow_dispatch: {}
permissions:
  contents: read
  pull-requests: read
concurrency:
  group: "update-github-info-${{ github.repository }}"
  cancel-in-progress: true
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
    max: 1
    fallback-as-issue: false
env:
  PR_TITLE: "Mona GitHub Info website update"
  PR_BRANCH_PREFIX: "auto/update-github-info"
jobs:
  preflight:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - id: preflight
        name: Preflight check for existing updater PR
        uses: actions/github-script@v9
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const core = require('@actions/core');
            const prPrefix = process.env.PR_BRANCH_PREFIX || 'auto/update-github-info';
            const owner = context.repo.owner;
            const repo = context.repo.repo;
            const prs = await github.paginate(github.rest.pulls.list, { owner, repo, state: 'open' });
            let found = false;
            let url = '';
            for (const p of prs) {
              if (p.head && p.head.ref && p.head.ref.startsWith(prPrefix)) { found = true; url = p.html_url; break; }
            }
            if (!found) {
              for (const p of prs) {
                const files = await github.paginate(github.rest.pulls.listFiles, { owner, repo, pull_number: p.number });
                if (files.some(f => f.filename === 'site/content/github-info.md')) { found = true; url = p.html_url; break; }
              }
            }
            core.setOutput('pr_found', found ? 'true' : 'false');
            core.setOutput('pr_url', url || '');
      - name: Exit early if updater PR found
        if: ${{ steps.preflight.outputs.pr_found == 'true' }}
        run: |
          echo "Updater PR already open: ${{ steps.preflight.outputs.pr_url }}"
          exit 0
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

## Important: Fetching URLs

Always use the built-in `web-fetch` tool for all URL fetching. Do NOT use `curl`, `wget`, or any shell command to fetch URLs — the raw HTML output is too large to process efficiently.

## Important: Patch Format Requirements

When using the `edit` tool to modify files, generate patches in this exact format:

```
*** Begin Patch
*** Update File: site/content/github-info.md
@@
 context line before change
-removed line
+added line
 context line after change
*** End Patch
```

The patch MUST start with `*** Begin Patch` and end with `*** End Patch`.
Use `*** Update File: {path}` as the file header — do NOT use `--- a/path` or `+++ b/path`.
Use `@@` to mark each hunk, followed by context lines (` `), removals (`-`), and additions (`+`).