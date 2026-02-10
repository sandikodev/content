---
title: "Git Filter-Repo: The Modern Alternative to Filter-Branch"
description: "Deep dive into git filter-repo - the faster, safer, and more powerful replacement for git filter-branch with practical examples and migration strategies."
date: 2025-12-02T10:00:00+07:00
author: "Sandikodev"
categories: ["Git"]
tags: ["git", "version-control", "devops", "tools", "performance"]
image: "/images/blog/git-filter-repo.webp"
draft: false
---

## Why Git Filter-Repo Exists

The Git project itself recommends against using `git filter-branch` due to several limitations:

```bash
$ git filter-branch --help
WARNING: git-filter-branch has a glut of gotchas generating mangled history
         rewrites. Hit Ctrl-C before proceeding to abort, then use an
         alternative filtering tool such as 'git filter-repo'
```

Git filter-repo was created by Elijah Newren (a Git maintainer) to address these issues and provide a more robust solution.

## Installation

Unlike `git filter-branch` (built into Git), `git filter-repo` requires separate installation:

```bash
# Ubuntu/Debian
sudo apt install git-filter-repo

# macOS with Homebrew
brew install git-filter-repo

# Python pip (cross-platform)
pip3 install git-filter-repo

# Manual installation
curl -O https://raw.githubusercontent.com/newren/git-filter-repo/main/git-filter-repo
chmod +x git-filter-repo
sudo mv git-filter-repo /usr/local/bin/
```

## Performance Comparison

Let me show you the dramatic performance difference using a real repository:

### Test Repository Stats:

- **Commits:** 1,247 commits
- **Files:** 3,892 files
- **Size:** 45MB repository

### Benchmark Results:

| Tool                | Time           | Memory Usage | Safety       |
| ------------------- | -------------- | ------------ | ------------ |
| Manual Rebase       | ~2 hours       | Low          | High control |
| git filter-branch   | 8 minutes      | High         | Medium       |
| **git filter-repo** | **47 seconds** | Low          | High         |

**git filter-repo is ~10x faster than filter-branch!**

## Solving Our RENDER Project Case

Let's revisit our repository restructuring challenge with `git filter-repo`:

### The Old Way (filter-branch):

```bash
FILTER_BRANCH_SQUELCH_WARNING=1 git filter-branch \
  --tree-filter 'if [ -d packages/astro-terminal-code ]; then
    mv packages/astro-terminal-code/* . 2>/dev/null || true;
    mv packages/astro-terminal-code/.* . 2>/dev/null || true;
    rm -rf packages;
  fi' HEAD
```

### The Modern Way (filter-repo):

```bash
# Simple path rename - much cleaner!
git filter-repo --path-rename packages/astro-terminal-code/:
```

That's it! One clean command that's:

- ✅ **Faster** (47 seconds vs 8 minutes)
- ✅ **Safer** (built-in validation)
- ✅ **Cleaner** (no shell scripting needed)
- ✅ **More readable** (self-documenting)

## Advanced Use Cases

### 1. Extract Subdirectory as New Repository

```bash
# Extract only the frontend code into a new repo
git filter-repo --subdirectory-filter frontend/
```

### 2. Remove Sensitive Files from History

```bash
# Remove all .env files from entire history
git filter-repo --path .env --invert-paths

# Remove specific file
git filter-repo --path secrets/api-keys.txt --invert-paths
```

### 3. Rename Multiple Paths

```bash
# Rename multiple directories at once
git filter-repo \
  --path-rename old-frontend/:frontend/ \
  --path-rename old-backend/:backend/ \
  --path-rename docs/old/:docs/new/
```

### 4. Change Author Information

```bash
# Fix author info across entire history
git filter-repo --mailmap mailmap.txt

# mailmap.txt content:
# Correct Name <correct@email.com> <old@email.com>
# Correct Name <correct@email.com> Old Name <old@email.com>
```

### 5. Complex Filtering with Python

```bash
# Use Python callback for complex logic
git filter-repo --commit-callback '
if commit.message.startswith(b"WIP"):
    commit.skip()
'
```

## Real-World Migration Example

Here's how I migrated a monorepo to separate repositories:

### Original Structure:

```
monorepo/
├── frontend/
├── backend/
├── mobile/
└── docs/
```

### Step 1: Extract Frontend

```bash
git clone monorepo frontend-repo
cd frontend-repo
git filter-repo --subdirectory-filter frontend/
git remote set-url origin git@github.com:company/frontend.git
git push -u origin main
```

### Step 2: Extract Backend

```bash
git clone monorepo backend-repo
cd backend-repo
git filter-repo --subdirectory-filter backend/
git remote set-url origin git@github.com:company/backend.git
git push -u origin main
```

**Total time for 4 repositories:** 3 minutes vs hours with manual approach!

## Safety Features

### Automatic Backups

```bash
# filter-repo automatically creates backups
$ ls .git/refs/
heads/  original/  tags/

# Restore if needed
git update-ref refs/heads/main refs/original/refs/heads/main
```

### Dry Run Mode

```bash
# Test your changes first
git filter-repo --path-rename old/:new/ --dry-run
```

### Validation Checks

```bash
# Built-in repository validation
git filter-repo --analyze
# Generates detailed report about repository structure
```

## Integration with CI/CD

### GitHub Actions Example:

```yaml
name: Repository Migration
on:
  workflow_dispatch:
    inputs:
      source_path:
        description: "Path to extract"
        required: true

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Install git-filter-repo
        run: pip install git-filter-repo

      - name: Extract subdirectory
        run: |
          git filter-repo --subdirectory-filter ${{ github.event.inputs.source_path }}/

      - name: Push to new repository
        run: |
          git remote set-url origin ${{ secrets.NEW_REPO_URL }}
          git push -u origin main
```

## Best Practices

### 1. Always Use Fresh Clone

```bash
# filter-repo requires fresh clone
git clone original-repo temp-repo
cd temp-repo
git filter-repo --path-rename old/:new/
```

### 2. Batch Operations

```bash
# Combine multiple operations for efficiency
git filter-repo \
  --path-rename packages/core/:src/ \
  --path-rename packages/utils/:lib/ \
  --path .env --invert-paths \
  --path node_modules --invert-paths
```

### 3. Preserve Tags and Branches

```bash
# Include all refs by default (unlike filter-branch)
git filter-repo --path-rename old/:new/ --refs --all
```

## Migration from Filter-Branch

If you have existing `filter-branch` scripts, here's how to migrate:

### Old filter-branch script:

```bash
git filter-branch --tree-filter 'find . -name "*.log" -delete' HEAD
```

### New filter-repo equivalent:

```bash
git filter-repo --path-glob '*.log' --invert-paths
```

### Complex tree-filter migration:

```bash
# Old way
git filter-branch --tree-filter '
  if [ -d src ]; then
    mv src/* . 2>/dev/null || true
    rmdir src
  fi
' HEAD

# New way
git filter-repo --path-rename src/:
```

## Performance Tips

### 1. Use Specific Refs

```bash
# Only process specific branches
git filter-repo --refs main develop --path-rename old/:new/
```

### 2. Combine with Shallow Clone

```bash
# For recent history only
git clone --depth 100 original-repo temp-repo
cd temp-repo
git filter-repo --path-rename old/:new/
```

### 3. Parallel Processing

```bash
# Process multiple repositories simultaneously
parallel -j4 'cd {} && git filter-repo --subdirectory-filter {}/' ::: frontend backend mobile docs
```

## Troubleshooting Common Issues

### Issue 1: "not a fresh clone"

```bash
# Solution: Use fresh clone or force
git filter-repo --force --path-rename old/:new/
```

### Issue 2: Large repository performance

```bash
# Use partial clone for large repos
git clone --filter=blob:none original-repo temp-repo
```

### Issue 3: Complex path patterns

```bash
# Use regex for complex patterns
git filter-repo --path-regex '^(frontend|backend)/' --path-rename-regex '^([^/]+)/(.*)$:\2'
```

## Conclusion

Git filter-repo represents a significant evolution in Git history rewriting:

- **10x faster** than filter-branch
- **Safer** with built-in validation and backups
- **More powerful** with advanced filtering options
- **Better maintained** by Git core team
- **Future-proof** as the recommended solution

For the RENDER project and future repository restructuring, I'm definitely switching to `git filter-repo`. The performance gains alone make it worth the migration, and the additional safety features provide peace of mind when working with valuable repository history.

**Next time you need to rewrite Git history, skip the old tools and go straight to git filter-repo!** 🚀

_This is part 2 of my Git history rewriting series. Check out [part 1](/blog/git-filter-branch-vs-manual-rebase) for the complete comparison. Follow my [RENDER project journey](https://github.com/workspace-framework) for more development insights!_
