# 🎯 Git Interview Questions

## 1. What is Git?

Git is a Distributed Version Control System (DVCS) used to track changes in source code and collaborate with developers.

---

## 2. What is Version Control?

Version Control is a system that records changes to files over time so previous versions can be restored.

---

## 3. What is the difference between Git and GitHub?

Git is a version control system.

GitHub is a cloud platform for hosting Git repositories.

---

## 4. Who created Git?

Linus Torvalds in 2005.

---

## 5. What is a Repository?

A repository is a storage location containing project files and complete version history.

---

## 6. What is the Working Directory?

The folder where developers modify project files.

---

## 7. What is the Staging Area?

A temporary area where changes are prepared before committing.

---

## 8. What is a Commit?

A commit is a snapshot of the project at a specific point in time.

---

## 9. What is git add?

Moves changes from the Working Directory to the Staging Area.

---

## 10. What is git commit?

Saves staged changes to the Local Repository.

---

## 11. What is git status?

Shows the current state of the repository.

---

## 12. What is git diff?

Displays changes made to tracked files.

---

## 13. What is git log?

Displays commit history.

---

## 14. What is git restore?

Discards local changes and restores the last committed version.

---

## 15. What is git rm?

Removes a tracked file from the repository.

---

## 16. What is git mv?

Renames or moves a tracked file.

---

## 17. What is .gitignore?

A file used to tell Git which files and directories should not be tracked.

---

## 18. What is the Git Workflow?

```
Working Directory
      ↓
Staging Area
      ↓
Local Repository
      ↓
Remote Repository
```

---

## 19. What are Git File States?

- Untracked
- Tracked
- Modified
- Staged
- Committed

---

## 20. Why is Git important for DevOps?

Git enables Infrastructure as Code (IaC), CI/CD pipelines, collaboration, rollback capabilities, and version control for code, configuration, Dockerfiles, Kubernetes manifests, and Terraform files.


# 🎯 Git Interview Questions & Answers

> Week 04 - Git & GitHub | Day 02

---

# 1. What is a Git Branch?

**Answer:**

A Git branch is an independent line of development that allows developers to work on new features, bug fixes, or experiments without affecting the main branch. It helps teams develop multiple features simultaneously.

---

# 2. Why do we use Git Branches?

**Answer:**

Git branches allow developers to:

- Work independently
- Develop multiple features in parallel
- Isolate code changes
- Keep the main branch stable
- Collaborate efficiently

---

# 3. What is the difference between `main` and `dev` branches?

**Answer:**

- **main** → Contains stable, production-ready code.
- **dev** → Used for ongoing development before changes are merged into the main branch.

---

# 4. What is Git Merge?

**Answer:**

Git Merge combines changes from one branch into another. It is commonly used to integrate completed feature branches into the main branch.

---

# 5. What is a Fast-Forward Merge?

**Answer:**

A Fast-Forward Merge occurs when the target branch has no new commits. Git simply moves the branch pointer forward without creating a merge commit.

---

# 6. What is a Three-Way Merge?

**Answer:**

A Three-Way Merge occurs when both branches have different commits. Git compares the common ancestor and creates a new merge commit to combine both histories.

---

# 7. What is a Merge Commit?

**Answer:**

A Merge Commit is a special commit created when Git merges two branches that have diverged. It has two parent commits and preserves the history of both branches.

---

# 8. What is a Merge Conflict?

**Answer:**

A Merge Conflict occurs when Git cannot automatically merge changes because the same part of a file has been modified differently in two branches.

---

# 9. How do you resolve a Merge Conflict?

**Answer:**

Steps:

1. Check the conflicting file using `git status`.
2. Open the file.
3. Remove conflict markers.
4. Keep the correct content.
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

# 10. What are Git Conflict Markers?

**Answer:**

Git inserts conflict markers inside the file:

```text
<<<<<<< HEAD
Current Branch
=======
Incoming Branch
>>>>>>> dev
```

These markers help identify conflicting changes and must be removed after resolving the conflict.

---

# 11. What is Git Stash?

**Answer:**

Git Stash temporarily stores uncommitted changes so that developers can switch branches or work on another task without creating an incomplete commit.

---

# 12. When should you use Git Stash?

**Answer:**

Git Stash is useful when:

- You need to switch branches.
- A production issue needs immediate attention.
- You don't want to commit unfinished work.

---

# 13. What is the difference between `git stash pop` and `git stash apply`?

| git stash pop | git stash apply |
|---------------|-----------------|
| Restores changes | Restores changes |
| Removes the stash | Keeps the stash |
| Used when stash is no longer needed | Used when stash may be reused |

---

# 14. What is Git Tag?

**Answer:**

A Git Tag is a permanent reference to a specific commit. Tags are commonly used to mark software releases such as `v1.0.0` or `v2.0.0`.

---

# 15. What are the types of Git Tags?

**Answer:**

### Lightweight Tag

A simple pointer to a commit.

Example:

```bash
git tag v1.0
```

### Annotated Tag

Stores additional metadata such as:

- Author
- Date
- Message

Example:

```bash
git tag -a v1.1 -m "First Stable Release"
```

---

# 16. What is Semantic Versioning?

**Answer:**

Semantic Versioning follows the format:

```
MAJOR.MINOR.PATCH
```

Example:

```
v2.3.1
```

- **MAJOR** → Breaking changes
- **MINOR** → New features
- **PATCH** → Bug fixes

---

# 17. Why are Git Tags important?

**Answer:**

Git Tags help developers:

- Mark software releases
- Roll back to previous versions
- Track stable releases
- Improve deployment management

---

# 18. Can a Git Tag move automatically?

**Answer:**

No.

A Git Tag always points to the same commit unless it is deleted and recreated.

Branches move with new commits, but tags remain fixed.

---

# 19. What are some Git Branching best practices?

**Answer:**

- Keep the `main` branch stable.
- Create a new branch for each feature.
- Merge only after testing.
- Delete merged branches.
- Use meaningful branch names.
- Pull the latest changes before merging.

---

# 20. How is Git used in a DevOps workflow?

**Answer:**

In DevOps, Git is used to:

- Manage infrastructure as code (Terraform, Ansible)
- Store Kubernetes manifests
- Track application source code
- Collaborate through feature branches
- Automate CI/CD pipelines
- Manage software releases using Git Tags

---

# ⭐ Quick Revision

### Branch Commands

```bash
git branch
git checkout -b dev
git switch dev
git branch -d dev
```

---

### Merge Commands

```bash
git merge dev
git log --graph --oneline --decorate --all
```

---

### Stash Commands

```bash
git stash
git stash list
git stash pop
git stash apply
```

---

### Tag Commands

```bash
git tag
git tag -a v1.0 -m "Release"
git show v1.0
git tag -d v1.0
```

---

# 📚 References

- Git Documentation: https://git-scm.com/doc
- Pro Git Book: https://git-scm.com/book/en/v2
- Atlassian Git Tutorials: https://www.atlassian.com/git/tutorials
- Learn Git Branching: https://learngitbranching.js.org/


