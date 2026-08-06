# ⚡ Git Cheatsheet

## Initialize Repository

```bash
git init
```

## Check Status

```bash
git status
```

## Add Files

```bash
git add .
```

## Commit

```bash
git commit -m "message"
```

## View History

```bash
git log --oneline
```

## Compare Changes

```bash
git diff
```

## Create Branch

```bash
git branch feature
```

## Switch Branch

```bash
git switch feature
```

## Merge Branch

```bash
git merge feature
```

## Push

```bash
git push origin main
```

## Pull

```bash
git pull origin main
```

## Clone Repository

```bash
git clone URL
```

## Restore Changes

```bash
git restore filename
```

## Remove File

```bash
git rm filename
```

## Rename File

```bash
git mv old new
```

## Stash

```bash
git stash
git stash pop
```

## View Branches

```bash
git branch
```

## View Remote

```bash
git remote -v
```

## Create Tag

```bash
git tag v1.0
```

---

## Git Workflow

```
Working Directory
      ↓
git add
      ↓
Staging Area
      ↓
git commit
      ↓
Local Repository
      ↓
git push
      ↓
GitHub
```

---

## File Lifecycle

```
Untracked
     ↓
git add
     ↓
Staged
     ↓
git commit
     ↓
Committed
     ↓
Modified
     ↓
git add
     ↓
Committed
```

# 📌 Git Cheatsheet

> Week 04 - Git & GitHub | Day 02

A quick reference for commonly used Git Branching, Merge, Stash, and Tag commands.

---

# 🌿 Git Branch Commands

## View all branches

### Syntax

```bash
git branch
```

### Example

```bash
git branch
```

### Purpose

Displays all local branches.

---

## Create a new branch

### Syntax

```bash
git branch <branch-name>
```

### Example

```bash
git branch dev
```

### Purpose

Creates a new branch without switching to it.

---

## Create and switch to a branch

### Syntax

```bash
git checkout -b <branch-name>
```

### Example

```bash
git checkout -b feature-login
```

### Purpose

Creates a new branch and switches to it immediately.

---

## Switch branches

### Syntax

```bash
git checkout <branch-name>
```

or

```bash
git switch <branch-name>
```

### Example

```bash
git checkout dev
```

### Purpose

Moves from the current branch to another branch.

---

## Rename current branch

### Syntax

```bash
git branch -M <new-name>
```

### Example

```bash
git branch -M main
```

### Purpose

Renames the current branch.

---

## Delete a merged branch

### Syntax

```bash
git branch -d <branch-name>
```

### Example

```bash
git branch -d dev
```

### Purpose

Deletes a branch that has already been merged.

---

## Force delete a branch

### Syntax

```bash
git branch -D <branch-name>
```

### Example

```bash
git branch -D feature-login
```

### Purpose

Deletes a branch even if it has not been merged.

---

# 🔀 Git Merge Commands

## Merge a branch

### Syntax

```bash
git merge <branch-name>
```

### Example

```bash
git merge dev
```

### Purpose

Combines another branch into the current branch.

---

## View commit graph

### Syntax

```bash
git log --graph --oneline --decorate --all
```

### Example

```bash
git log --graph --oneline --decorate --all
```

### Purpose

Displays branch history in graph format.

---

# ⚔️ Merge Conflict Commands

## Check repository status

### Syntax

```bash
git status
```

### Purpose

Shows modified, staged, and conflicted files.

---

## Stage resolved file

### Syntax

```bash
git add <file-name>
```

### Example

```bash
git add README.md
```

### Purpose

Marks the conflict as resolved.

---

## Complete merge

### Syntax

```bash
git commit
```

or

```bash
git commit -m "Resolve merge conflict"
```

### Purpose

Creates the merge commit after conflict resolution.

---

# 📦 Git Stash Commands

## Save current changes

### Syntax

```bash
git stash
```

### Purpose

Temporarily stores uncommitted changes.

---

## View saved stashes

### Syntax

```bash
git stash list
```

### Purpose

Displays all saved stashes.

---

## Restore latest stash

### Syntax

```bash
git stash pop
```

### Purpose

Restores and removes the latest stash.

---

## Restore stash without deleting

### Syntax

```bash
git stash apply
```

### Purpose

Restores the stash but keeps it in the stash list.

---

## Delete one stash

### Syntax

```bash
git stash drop stash@{0}
```

### Purpose

Deletes a specific stash.

---

## Delete all stashes

### Syntax

```bash
git stash clear
```

### Purpose

Removes every saved stash.

---

# 🏷️ Git Tag Commands

## View tags

### Syntax

```bash
git tag
```

### Purpose

Lists all tags.

---

## Create a lightweight tag

### Syntax

```bash
git tag <tag-name>
```

### Example

```bash
git tag v1.0
```

### Purpose

Creates a simple tag pointing to the current commit.

---

## Create an annotated tag

### Syntax

```bash
git tag -a <tag-name> -m "<message>"
```

### Example

```bash
git tag -a v1.1 -m "First Stable Release"
```

### Purpose

Creates a tag with author, date, and message.

---

## Show tag details

### Syntax

```bash
git show <tag-name>
```

### Example

```bash
git show v1.1
```

### Purpose

Displays detailed information about a tag.

---

## Tag an older commit

### Syntax

```bash
git tag <tag-name> <commit-hash>
```

### Example

```bash
git tag v0.9 b5f7352
```

### Purpose

Creates a tag for a previous commit.

---

## Delete a tag

### Syntax

```bash
git tag -d <tag-name>
```

### Example

```bash
git tag -d v0.9
```

### Purpose

Deletes a local tag.

---

# 📚 Common Git Workflow

```text
Create Branch
      │
      ▼
git checkout -b feature-login
      │
      ▼
Make Changes
      │
      ▼
git add .
      │
      ▼
git commit -m "Add login feature"
      │
      ▼
git checkout main
      │
      ▼
git merge feature-login
```

---

# 📦 Git Stash Workflow

```text
Modify Files
      │
      ▼
git stash
      │
      ▼
Working Directory Clean
      │
      ▼
Switch Branch
      │
      ▼
git stash pop
      │
      ▼
Continue Working
```

---

# 🏷️ Git Tag Workflow

```text
Develop Feature
      │
      ▼
Commit Changes
      │
      ▼
Create Release Tag
      │
      ▼
git tag -a v1.0 -m "Release"
```

---

# 💡 Best Practices

- ✅ Create a new branch for every feature.
- ✅ Keep the `main` branch stable.
- ✅ Write meaningful commit messages.
- ✅ Pull the latest changes before merging.
- ✅ Resolve merge conflicts carefully.
- ✅ Use `git stash` for temporary work.
- ✅ Tag production releases.
- ✅ Follow Semantic Versioning (`v1.0.0`).

---

# 🚀 Quick Revision

| Task | Command |
|------|---------|
| View Branches | `git branch` |
| Create Branch | `git branch dev` |
| Create & Switch | `git checkout -b dev` |
| Switch Branch | `git checkout dev` |
| Merge Branch | `git merge dev` |
| View Graph | `git log --graph --oneline --decorate --all` |
| Check Status | `git status` |
| Save Work | `git stash` |
| Restore Work | `git stash pop` |
| List Stashes | `git stash list` |
| Create Tag | `git tag v1.0` |
| Annotated Tag | `git tag -a v1.1 -m "Release"` |
| Show Tag | `git show v1.1` |
| Delete Tag | `git tag -d v1.0` |

---

## 📖 Useful Resources

- Git Official Docs: https://git-scm.com/doc
- Pro Git Book: https://git-scm.com/book/en/v2
- Learn Git Branching: https://learngitbranching.js.org/
- Atlassian Git Tutorials: https://www.atlassian.com/git/tutorials
- GitHub Skills: https://skills.github.com/


✓ Pull before push

✓ Tag releases

✓ Use stash for temporary work
