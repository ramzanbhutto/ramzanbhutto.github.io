---
title: "Git and Github: A Comprehensive Guide"
date: "2026-02-09"
draft: false
showToc: true
Author: "Muhammad Ramzan"
tags: ["git", "github", "version control", "collaboration"]
description: "A comprehensive guide to using Git and GitHub for version control and collaboration in software development."
---

## Introduction

![Git & GitHub](/git.png)
Git is a version control system that tracks changes in code and helps developers collaborate efficiently. GitHub is an online platform that hosts Git repositories, making it easier to share code, manage projects, and work on open-source projects.

---

## Getting Started

### Initial Configuration

```bash
git config --global user.name "your-name"
git config --global user.email "your-email"
git config --global core.editor "code"  # Set default editor
git config --list  # View all configurations
```

### Creating Repositories

```bash
git init  # Initialize a new repository
git clone <repo-url>  # Clone an existing repository
git clone <repo-url> <folder-name>  # Clone into specific folder
```

---

## Basic Workflow

### Staging Changes

```bash
git status  # Check current status
git add <file>  # Stage a specific file
git add .  # Stage all files
git add *.js  # Stage all JavaScript files
git rm <file>  # Remove file and stage deletion
git mv <old-name> <new-name>  # Rename file and stage change
```

### Committing Changes

```bash
git commit -m "Describe your changes here"
git commit -am "Add and commit tracked files"
git commit --amend -m "Update last commit message"
git commit --amend --no-edit  # Add staged changes to last commit
```

### Viewing History

```bash
git log  # View commit history
git log --oneline  # Compact commit history
git log --graph --oneline --all  # Visual branch history
git log -p  # Show changes in each commit
git log -n 5  # Show last 5 commits
git show <commit-hash>  # Show specific commit details
git diff  # Show unstaged changes
git diff --staged  # Show staged changes
git diff <branch1> <branch2>  # Compare branches
```

---

## Branching & Merging

### Managing Branches

```bash
git branch  # List all local branches
git branch -a  # List all branches (local and remote)
git branch <branch-name>  # Create a new branch
git checkout <branch-name>  # Switch to a branch
git checkout -b <branch-name>  # Create and switch to new branch
git switch <branch-name>  # Modern way to switch branches
git switch -c <branch-name>  # Create and switch to new branch
git branch -d <branch-name>  # Delete branch (safe)
git branch -D <branch-name>  # Force delete branch
git branch -m <new-name>  # Rename current branch
```

### Merging

```bash
git merge <branch-name>  # Merge branch into current branch
git merge --no-ff <branch-name>  # Merge with merge commit
git merge --squash <branch-name>  # Squash commits before merging
git merge --abort  # Abort merge in case of conflicts
```

### Rebasing

```bash
git rebase <branch-name>  # Rebase current branch
git rebase -i HEAD~3  # Interactive rebase last 3 commits
git rebase --continue  # Continue after resolving conflicts
git rebase --abort  # Abort rebase operation
```

---

## Working with Remotes

### Remote Management

```bash
git remote  # List remote connections
git remote -v  # List remotes with URLs
git remote add origin <repo-url>  # Add remote repository
git remote set-url origin <new-url>  # Change remote URL
git remote rename <old-name> <new-name>  # Rename remote
git remote remove <remote-name>  # Remove remote
git remote show origin  # Show remote details
```

### Syncing with Remotes

```bash
git fetch  # Download remote changes (no merge)
git fetch origin  # Fetch from specific remote
git fetch --all  # Fetch from all remotes
git pull  # Fetch and merge remote changes
git pull --rebase  # Fetch and rebase instead of merge
git push  # Push changes to remote
git push -u origin <branch-name>  # Push and set upstream
git push origin --delete <branch-name>  # Delete remote branch
git push --force  # Force push (use with caution!)
git push --force-with-lease  # Safer force push
git push --tags  # Push all tags to remote
```

---

## Undoing Changes

### Working Directory & Staging

```bash
git checkout -- <file>  # Discard changes in working directory
git restore <file>  # Modern way to discard changes
git restore --staged <file>  # Unstage file
git reset <file>  # Unstage file (keep changes)
git reset --hard  # Discard all local changes
git clean -fd  # Remove untracked files and directories
git clean -n  # Preview files to be removed
```

### Commits

```bash
git reset --soft HEAD~1  # Undo last commit (keep changes staged)
git reset --mixed HEAD~1  # Undo last commit (keep changes unstaged)
git reset --hard HEAD~1  # Undo last commit (discard changes)
git reset --hard <commit-hash>  # Reset to specific commit
git revert <commit-hash>  # Create new commit that undoes changes
git revert HEAD  # Revert last commit
```

---

## Stashing

### Temporary Storage

```bash
git stash  # Stash current changes
git stash save "message"  # Stash with description
git stash list  # List all stashes
git stash show  # Show latest stash changes
git stash show stash@{0}  # Show specific stash
git stash apply  # Apply latest stash (keep in list)
git stash apply stash@{2}  # Apply specific stash
git stash pop  # Apply latest stash and remove from list
git stash drop  # Delete latest stash
git stash drop stash@{1}  # Delete specific stash
git stash clear  # Delete all stashes
git stash branch <branch-name>  # Create branch from stash
```

---

## Tagging

### Version Marking

```bash
git tag  # List all tags
git tag <tag-name>  # Create lightweight tag
git tag -a v1.0 -m "Version 1.0"  # Create annotated tag
git tag -a v1.0 <commit-hash> -m "Tag old commit"  # Tag specific commit
git show <tag-name>  # Show tag details
git push origin <tag-name>  # Push specific tag
git push origin --tags  # Push all tags
git tag -d <tag-name>  # Delete local tag
git push origin --delete <tag-name>  # Delete remote tag
```

---

## Advanced Features

### Submodules

```bash
git submodule add <repo-url> <path>  # Add submodule
git submodule init  # Initialize submodules
git submodule update  # Update submodules
git submodule update --init --recursive  # Initialize and update all
git submodule foreach git pull origin main  # Update all submodules
git submodule deinit <path>  # Deinitialize submodule
git rm <submodule-path>  # Remove submodule
```

### Cherry-Picking

```bash
git cherry-pick <commit-hash>  # Apply specific commit to current branch
git cherry-pick <hash1> <hash2>  # Apply multiple commits
git cherry-pick --continue  # Continue after resolving conflicts
git cherry-pick --abort  # Abort cherry-pick operation
```

### Searching & Finding

```bash
git grep "search-term"  # Search in working directory
git grep "search-term" <branch-name>  # Search in specific branch
git log -S "search-term"  # Find commits with specific content
git log --grep="pattern"  # Search commit messages
git bisect start  # Start binary search for bug
git bisect bad  # Mark current commit as bad
git bisect good <commit-hash>  # Mark commit as good
git bisect reset  # End bisect session
git blame <file>  # Show who changed each line
git reflog  # Show reference logs (recover lost commits)
```

### Aliases

```bash
git config --global alias.co checkout  # Create alias
git config --global alias.br branch
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --graph --oneline --all'
```

---

## GitHub-Specific Features

### Pull Requests & Collaboration

```bash
# Fetch PR for local testing
git fetch origin pull/123/head:pr-123
git checkout pr-123

# Working with forks
git remote add upstream <original-repo-url>
git fetch upstream
git merge upstream/main
```

### GitHub CLI (gh)

```bash
gh repo clone <repo-name>  # Clone repository
gh pr create  # Create pull request
gh pr list  # List pull requests
gh pr checkout <pr-number>  # Checkout pull request
gh issue create  # Create issue
gh issue list  # List issues
```

---

## Practical Example: Push a Project to GitHub

1. **Configure Git:**
```bash
git config --global user.name "your-name"
git config --global user.email "your-email"
```

2. **Initialize repository:**
```bash
git init
```

3. **Link to GitHub:**
```bash
git remote add origin <repo-url>
```

4. **Stage and commit:**
```bash
git add .
git commit -m "Initial commit"
```

5. **Push to GitHub:**
```bash
git push -u origin main
```

Your project is now live on GitHub!

---

## Troubleshooting

### Common Issues

```bash
# Undo accidental commit to wrong branch
git reset --soft HEAD~1
git stash
git checkout correct-branch
git stash pop
git commit -m "message"

# Fix merge conflicts
git status  # Identify conflicted files
# Edit files to resolve conflicts
git add <resolved-files>
git commit

# Sync fork with upstream
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Recover deleted branch
git reflog
git checkout -b <branch-name> <commit-hash>
```

---

## Best Practices

- **Commit often** with clear, descriptive messages
- **Pull before push** to avoid conflicts
- **Use branches** for features and bug fixes
- **Never force push** to shared branches
- **Review changes** before committing (`git diff`)
- **Keep commits atomic** (one logical change per commit)
- **Write meaningful commit messages** (what and why, not how)
- **Use `.gitignore`** to exclude unnecessary files
- **Tag releases** for version management
- **Protect main branch** on GitHub

---
