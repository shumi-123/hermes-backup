---
name: github-ssh-wsl
description: Configure GitHub SSH access from WSL for git push/pull, avoiding API call blocks
version: 1.0.0
tags: [github, ssh, wsl, git, backup]
---

# GitHub SSH Setup from WSL

## When to Use
Configure GitHub SSH access from WSL for git push/pull operations.

## Steps

### 1. Generate SSH Key
```bash
ssh-keygen -t ed25519 -C "your@email" -f ~/.ssh/id_ed25519 -N ""
```

### 2. Add to GitHub
Copy the public key:
```bash
cat ~/.ssh/id_ed25519.pub
```
Go to https://github.com/settings/keys → New SSH key → paste

### 3. Verify SSH Access
```bash
ssh-keyscan github.com >> ~/.ssh/known_hosts 2>/dev/null
ssh -T git@github.com
```
Expected: `Hi <username>! You've successfully authenticated`

### 4. Use SSH Remote URLs
```bash
git remote add origin git@github.com:username/repo.git
git push -u origin master
```

## Common WSL Issues

| Problem | Solution |
|---------|----------|
| `Host key verification failed` | Run `ssh-keyscan github.com >> ~/.ssh/known_hosts` |
| `Permission denied (publickey)` | Windows and WSL SSH are independent; add key to GitHub |
| API calls blocked (curl/Python urllib) | Use SSH git operations instead of GitHub API |

## Key Insight
API calls (curl, Python urllib, gh CLI) are often blocked by security scanning in WSL environments. SSH git operations work reliably. Create repositories via browser at https://github.com/new instead of using the API.
