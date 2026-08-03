# 💻 Git Commands Reference

This document contains the most commonly used Git commands with their descriptions and examples.

---

# Check Git Version

```bash
git --version
```

Displays the installed Git version.

---

# Configure Git

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

Configure Git username and email globally.

---

# View Git Configuration

```bash
git config --list
```

Shows all configured Git settings.

---

# Initialize Repository

```bash
git init
```

Creates a new Git repository.

---

# Repository Status

```bash
git status
```

Displays the current state of the repository.

---

# Add Files

Add a specific file

```bash
git add filename
```

Add all files

```bash
git add .
```

---

# Commit Changes

```bash
git commit -m "Commit Message"
```

Creates a snapshot of staged changes.

---

# View Commit History

```bash
git log
```

Compact view

```bash
git log --oneline
```

---

# Compare Changes

```bash
git diff
```

Shows file differences before staging.

---

# Restore File

```bash
git restore filename
```

Discard local changes.

---

# Remove File

```bash
git rm filename
```

Deletes a tracked file.

---

# Rename File

```bash
git mv oldname newname
```

Renames a tracked file.

---

# Branch Information

```bash
git branch
```

List all branches.

---

# Create Branch

```bash
git branch feature-name
```

---

# Switch Branch

```bash
git switch feature-name
```

or

```bash
git checkout feature-name
```

---

# Merge Branch

```bash
git merge feature-name
```

---

# View Remote

```bash
git remote -v
```

---

# Push Changes

```bash
git push origin main
```

---

# Pull Changes

```bash
git pull origin main
```

---

# Clone Repository

```bash
git clone <repository-url>
```

---

# Fetch Changes

```bash
git fetch
```

---

# Stash Changes

```bash
git stash
git stash pop
```

---

# Tags

```bash
git tag
```

Create a tag

```bash
git tag v1.0
```

---

# Useful Commands

```bash
git status
git log --oneline
git diff
git branch
git remote -v
```
