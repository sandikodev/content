---
title: "Git Filter-Branch vs Manual Rebase: Restructuring Repository History"
description: "Comparing two approaches to restructure Git repository history - the powerful git filter-branch command versus manual rebase strategy for moving files across commits."
date: 2025-10-28T10:00:00+07:00
author: "Sandikodev"
categories: ["Git"]
tags: ["git", "version-control", "devops", "workflow"]
image: "/images/blog/git-filter-branch.webp"
ai_features:
  mermaid_diagrams: false
  code_snippets: true
  technical_depth: "intermediate"
content_type: "technical"
auto_toc: true
reading_time: true
draft: false
---

## The Problem

Recently, while working on the [RENDER project](https://github.com/workspace-framework), I had a repository with this structure:

```
terminal-code/
├── packages/
│   └── astro-terminal-code/
│       ├── package.json
│       ├── README.md
│       └── src/
```

But I needed it to be:

```
terminal-code/
├── package.json
├── README.md
└── src/
```

The challenge? I wanted to preserve the entire commit history while restructuring the files across **all commits**, not just the latest one.

## Approach 1: Manual Rebase Strategy

My usual approach for this type of problem involves manually going through each commit:

```bash
# 1. Start interactive rebase from root
git rebase -i --root

# 2. Mark each commit as 'edit'
# 3. For each commit, manually:
git reset --soft HEAD~1
# Move files manually
mv packages/astro-terminal-code/* .
rm -rf packages/
git add -A
git commit --amend --no-edit

# 4. Continue to next commit
git rebase --continue

# 5. Repeat until all commits are processed
```

### Pros of Manual Rebase:

- ✅ Full control over each commit
- ✅ Can make different changes per commit if needed
- ✅ Easy to understand what's happening
- ✅ Can abort/fix issues at any step

### Cons of Manual Rebase:

- ❌ Time-consuming for many commits
- ❌ Prone to human error
- ❌ Interactive editor issues in some environments
- ❌ Repetitive and tedious process

## Approach 2: Git Filter-Branch

Then I discovered `git filter-branch` - a powerful command that can rewrite entire repository history in one go:

```bash
# Single command to restructure entire history
FILTER_BRANCH_SQUELCH_WARNING=1 git filter-branch \
  --tree-filter 'if [ -d packages/astro-terminal-code ]; then
    mv packages/astro-terminal-code/* . 2>/dev/null || true;
    mv packages/astro-terminal-code/.* . 2>/dev/null || true;
    rm -rf packages;
  fi' HEAD
```

### Pros of Filter-Branch:

- ✅ **One command** handles entire history
- ✅ **Consistent** changes across all commits
- ✅ **Fast** execution
- ✅ **No interactive editor** issues
- ✅ **Atomic operation** - all or nothing

### Cons of Filter-Branch:

- ❌ Less granular control
- ❌ Can be dangerous if command is wrong
- ❌ Harder to debug if something goes wrong
- ❌ Rewrites all commit hashes

## Real-World Comparison

Let me show you the actual results from my RENDER project restructuring:

### Before:

```bash
$ git log --oneline
65e77b3 🔥 Update branding: SandikoOS → RENDER
a2219f5 🚀 FOUNDATION: SandikoOS Ecosystem - Phase 1

$ git show --name-only a2219f5
packages/astro-terminal-code/README.md
packages/astro-terminal-code/package.json
packages/astro-terminal-code/src/index.ts
packages/astro-terminal-code/src/processor.ts
```

### After Filter-Branch:

```bash
$ git log --oneline
31a26da 🔥 Update branding: SandikoOS → RENDER
064115d 🚀 FOUNDATION: SandikoOS Ecosystem - Phase 1

$ git show --name-only 064115d
README.md
package.json
src/index.ts
src/processor.ts
```

**Time taken:**

- Manual rebase: ~10-15 minutes (with interactive editor struggles)
- Filter-branch: ~5 seconds

## When to Use Each Approach

### Use Manual Rebase When:

- You need different changes per commit
- You want to review each commit individually
- You're learning Git and want to understand the process
- You have complex merge conflicts to resolve

### Use Filter-Branch When:

- You need consistent changes across all commits
- You have many commits to process
- You want a fast, automated solution
- You're confident in your filter command

## Modern Alternative: Git Filter-Repo

For new projects, consider `git filter-repo` (requires separate installation):

```bash
# More modern and safer alternative
git filter-repo --path-rename packages/astro-terminal-code/:
```

## Safety Tips

Regardless of which approach you choose:

1. **Always backup** your repository first
2. **Test on a clone** before applying to main repo
3. **Use `--force-with-lease`** when pushing rewritten history
4. **Coordinate with team** before rewriting shared history

```bash
# Safe backup and testing workflow
git clone original-repo test-repo
cd test-repo
# Apply your changes
git push --force-with-lease origin main
```

## Conclusion

Both approaches have their place in a developer's toolkit. For the RENDER project restructuring, `git filter-branch` was clearly the winner - it saved me significant time and eliminated the interactive editor issues I was facing.

However, the manual rebase approach taught me a lot about Git internals and gives me confidence to handle complex history rewriting scenarios.

**My recommendation:** Start with understanding the manual approach, then graduate to `git filter-branch` for efficiency. Your future self will thank you for learning both! 🚀

_This article is part of my journey building the [RENDER ecosystem](https://github.com/workspace-framework) - revolutionizing desktop development with web technologies. Follow along for more Git tips and development insights!_
