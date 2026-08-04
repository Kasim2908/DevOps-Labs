# 📝 Day 01 - Git Fundamentals | Learning Notes

## 📅 Date

03 August 2026

---

# 📚 Topics Covered

Today I learned the fundamentals of Git and GitHub, including Version Control concepts, Git architecture, Git workflow, and essential Git commands.

---

# 📖 Theory

## ✅ What is Version Control?

A Version Control System (VCS) tracks changes made to files over time. It allows developers to:

- Track file history
- Restore previous versions
- Collaborate with team members
- Compare file changes
- Prevent accidental data loss

Think of Git as a **time machine for your code**.

---

## ✅ Types of Version Control Systems

### Local Version Control System (LVCS)

- Stores project history on a single computer.
- Suitable for individual work.
- No collaboration.

### Centralized Version Control System (CVCS)

Examples:

- SVN
- CVS

Features:

- Single central server.
- Multiple developers connect to one repository.
- Server failure affects everyone.

### Distributed Version Control System (DVCS)

Examples:

- Git
- Mercurial

Features:

- Every developer has a complete copy of the repository.
- Offline support.
- Faster operations.
- Better collaboration.

Git is a **Distributed Version Control System (DVCS).**

---

## ✅ What is Git?

Git is a distributed version control system created by **Linus Torvalds** in **2005**.

Git allows developers to:

- Track code changes
- Create commits
- Manage branches
- Merge code
- Restore previous versions
- Collaborate efficiently

---

## ✅ Git vs GitHub

| Git | GitHub |
|------|---------|
| Version Control System | Git Repository Hosting Platform |
| Works Locally | Cloud Platform |
| Offline Support | Online Collaboration |
| Tracks Changes | Hosts Repositories |

---

## ✅ Git Architecture

Git manages changes through four stages:

```text
Working Directory
       │
       ▼
Staging Area
       │
       ▼
Local Repository
       │
       ▼
Remote Repository (GitHub)
```

---

## ✅ Git File Lifecycle

```text
Untracked

↓

Staged

↓

Committed

↓

Modified

↓

Staged

↓

Committed
```

---

# 💻 Hands-on Commands Practiced

```bash
git --version

git config --list

git init

git status

git add

git commit -m "message"

git log

git log --oneline

git diff

git restore

git rm

git mv
```

---

# 🛠 Practical Activities

- Installed and verified Git.
- Configured Git username and email.
- Created a Git repository.
- Understood the `.git` directory.
- Created new files.
- Checked repository status.
- Staged files.
- Created commits.
- Viewed commit history.
- Compared file changes.
- Restored modified files.
- Removed tracked files.
- Renamed tracked files.
- Created a `.gitignore` file.

---

# ⚠️ Common Mistakes Learned

### 1. Forgetting `-m` while committing

❌

```bash
git commit "Initial Commit"
```

✅

```bash
git commit -m "Initial Commit"
```

---

### 2. Using `git rm` on an untracked file

❌

```bash
git rm temp.txt
```

Git only removes **tracked files**.

---

### 3. Using `git mv` on an untracked file

A file must be committed before it can be renamed using Git.

---

### 4. Typographical Errors

Example:

```bash
git rv
```

Correct:

```bash
git mv
```

---

# 🎯 Key Takeaways

- Git tracks project history.
- Every commit acts as a save point.
- The Staging Area allows selective commits.
- Git works offline.
- GitHub is used for collaboration and repository hosting.
- `git status` is one of the most important Git commands.
- Small, meaningful commits make project history easier to understand.

---

# 📌 Commands to Remember

```bash
git init

git status

git add .

git commit -m "message"

git log

git log --oneline

git diff

git restore

git rm

git mv
```

---

# 🚀 What's Next?

In the next session, I will learn:

- Git Branches
- Git Switch
- Git Checkout
- Git Merge
- Merge Conflicts
- Git Stash
- Git Tags
- GitHub Remotes
- Git Pull & Push
- Pull Requests
- Team Collaboration Workflow

# 🌿 Git Branching, Merging, Stash & Tags

> Week 04 - Git & GitHub | Day 02 Notes

---

# 1. Git Branching

## What is a Git Branch?

A Git branch is an independent line of development. It allows developers to work on new features, bug fixes, or experiments without affecting the main project.

Each branch has its own commit history until it is merged.

### Why Branches?

- Multiple developers can work simultaneously.
- Keeps the main branch stable.
- Allows safe experimentation.
- Makes collaboration easier.

### Common Branches

- `main` → Stable production code
- `dev` → Development branch
- `feature/*` → New feature development
- `bugfix/*` → Bug fixes
- `hotfix/*` → Emergency production fixes

---

# 2. Git Branch Commands

## View Branches

```bash
git branch
```

Displays all local branches.

---

## Create a Branch

```bash
git branch dev
```

Creates a new branch named **dev**.

---

## Create and Switch

```bash
git checkout -b dev
```

Creates the branch and immediately switches to it.

---

## Switch Branch

```bash
git checkout dev
```

or

```bash
git switch dev
```

Moves from the current branch to another branch.

---

## Delete Branch

```bash
git branch -d dev
```

Deletes a merged branch.

Force delete:

```bash
git branch -D dev
```

---

# 3. Git Merge

## What is Merge?

Git Merge combines changes from one branch into another.

Example:

```
feature-login

↓

Completed

↓

Merged into main
```

This allows completed work to become part of the main project.

---

# 4. Types of Merge

## Fast-Forward Merge

A Fast-Forward Merge happens when the target branch has no additional commits.

Git simply moves the branch pointer forward.

Example:

```
main

↓

Commit A

↓

Commit B (dev)

↓

Fast Forward

↓

main & dev
```

### Advantages

- No merge commit created
- Clean Git history
- Simple workflow

---

## Three-Way Merge

A Three-Way Merge happens when both branches have different commits.

Git creates a new merge commit that combines both histories.

Example:

```
          dev
           │
           C
          /
A ---- B
          \
           D
           │
        main
```

### Advantages

- Preserves complete history
- Better for team collaboration
- Shows when branches were merged

---

# 5. Merge Commit

A Merge Commit is a special commit created after a Three-Way Merge.

It has two parent commits and combines both branch histories.

---

# 6. Merge Conflict

## What is a Merge Conflict?

A Merge Conflict occurs when Git cannot automatically merge changes because the same part of a file has been modified differently in two branches.

Git asks the developer to resolve the conflict manually.

Example:

Developer A

```
Hello World
```

Developer B

```
Hello Git
```

Git cannot decide which version should be kept.

---

# 7. Conflict Markers

When a conflict occurs, Git inserts markers inside the file.

Example:

```text
<<<<<<< HEAD
This change is from MASTER
=======
This change is from DEV
>>>>>>> dev
```

Meaning:

- `<<<<<<< HEAD` → Current branch changes
- `=======` → Separator
- `>>>>>>> dev` → Incoming branch changes

Remove these markers after resolving the conflict.

---

# 8. Resolving Merge Conflicts

Steps:

1. Open the conflicting file.
2. Review both versions.
3. Keep the correct content.
4. Remove conflict markers.
5. Save the file.
6. Stage the file.

```bash
git add README.md
```

7. Complete the merge.

```bash
git commit
```

---

# 9. Git Stash

## What is Git Stash?

Git Stash temporarily saves uncommitted changes and restores the working directory to the last committed state.

It is useful when you need to switch tasks without creating an incomplete commit.

### Example

You are working on a feature.

Suddenly a production issue occurs.

Instead of committing unfinished code:

```bash
git stash
```

Switch branches, fix the issue, then restore your work.

---

# 10. Git Stash Workflow

```
Modify Files

↓

git stash

↓

Temporary Storage

↓

Working Directory Clean

↓

git stash pop

↓

Restore Changes
```

---

# 11. Git Stash Commands

Save changes

```bash
git stash
```

View saved stashes

```bash
git stash list
```

Restore latest stash

```bash
git stash pop
```

Restore without deleting

```bash
git stash apply
```

Delete one stash

```bash
git stash drop stash@{0}
```

Delete all stashes

```bash
git stash clear
```

---

# 12. Git Tags

## What is a Git Tag?

A Git Tag is a permanent label attached to a specific commit.

Tags are commonly used to mark software releases.

Example:

```
Commit 1

Commit 2

🏷 v1.0

Commit 3

🏷 v2.0
```

---

# 13. Types of Git Tags

## Lightweight Tag

Simple pointer to a commit.

```bash
git tag v1.0
```

---

## Annotated Tag

Stores additional information like author, date, and message.

```bash
git tag -a v1.1 -m "First Stable Release"
```

Preferred for production releases.

---

# 14. Git Tag Commands

View tags

```bash
git tag
```

Create lightweight tag

```bash
git tag v1.0
```

Create annotated tag

```bash
git tag -a v1.1 -m "Release"
```

View tag details

```bash
git show v1.1
```

Tag an older commit

```bash
git tag v0.9 <commit-hash>
```

Delete a tag

```bash
git tag -d v0.9
```

---

# 15. Semantic Versioning

Most software follows:

```
MAJOR.MINOR.PATCH
```

Examples:

```
v1.0.0

v1.1.0

v1.1.1

v2.0.0
```

Meaning:

- **MAJOR** → Breaking changes
- **MINOR** → New features
- **PATCH** → Bug fixes

---

# 16. Real DevOps Use Cases

### Git Branches

- `feature-monitoring`
- `feature-terraform`
- `hotfix-production`

Developers work independently and merge only after testing.

---

### Git Stash

Temporarily save Terraform or Kubernetes changes while fixing an urgent production issue.

---

### Git Tags

Deploy applications using versioned Docker images.

Example:

```yaml
image: myapp:v2.1.0
```

Instead of:

```yaml
image: myapp:latest
```

Versioned tags make deployments reproducible and simplify rollbacks.

---

# 17. Best Practices

- Keep the `main` branch stable.
- Create a separate branch for every feature.
- Write meaningful commit messages.
- Pull the latest changes before merging.
- Resolve merge conflicts carefully.
- Use `git stash` instead of committing incomplete work.
- Tag production releases.
- Follow Semantic Versioning.
- Review changes before pushing.

---

# 18. Key Takeaways

✔ Git Branches enable parallel development.

✔ Fast-Forward Merge moves the branch pointer without creating a merge commit.

✔ Three-Way Merge creates a merge commit when branches have diverged.

✔ Merge Conflicts occur when Git cannot automatically combine changes.

✔ Git Stash temporarily stores unfinished work.

✔ Git Tags mark important commits such as software releases.

✔ Semantic Versioning helps maintain consistent software release versions.

