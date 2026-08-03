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
