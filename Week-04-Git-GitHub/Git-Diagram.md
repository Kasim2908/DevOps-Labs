# 📊 Git Diagrams

> Week 04 - Git & GitHub

A collection of important Git diagrams to understand Git concepts visually.

---

# 1. Git Architecture

```text
                Working Directory
                       │
              git add  │
                       ▼
                Staging Area
                       │
           git commit  │
                       ▼
              Local Repository
                       │
        git push / pull│
                       ▼
             Remote Repository
                  (GitHub)
```

---

# 2. Git File Lifecycle

```text
          Create File
               │
               ▼
          Untracked File
               │
         git add file
               ▼
           Staged File
               │
     git commit -m "msg"
               ▼
        Tracked File
               │
      Modify the File
               ▼
        Modified File
               │
          git add
               ▼
        Staged Again
               │
        git commit
               ▼
      Updated Version
```

---

# 3. Git Branching

```text
                  main
                    │
         ┌──────────┼──────────┐
         │          │          │
     feature-A  feature-B  feature-C
         │          │          │
         └──────────┼──────────┘
                    │
                 Merge
                    │
                  main
```

---

# 4. Branch Creation

```text
Initial Commit
      │
      ▼
     main
      │
      ▼
git checkout -b dev
      │
      ▼
      main
        \
         \
          dev
```

---

# 5. Fast-Forward Merge

```text
Before Merge

main
 │
 A
 │
 B
 │
 C (dev)

After Merge

main
 │
 A
 │
 B
 │
 C
 ▲
 │
dev
```

Git simply moves the branch pointer.

---

# 6. Three-Way Merge

```text
             dev
              │
              C
             /
A ---------- B
             \
              D
              │
            main

↓

Merge

             Merge Commit
             /         \
            C           D
             \         /
               A ---- B
```

Git creates a new merge commit.

---

# 7. Merge Conflict

```text
master

README.md

Hello World

↓

dev

README.md

Hello Git

↓

Git Cannot Decide

↓

Merge Conflict
```

---

# 8. Conflict Markers

```text
<<<<<<< HEAD
This change is from MASTER
=======
This change is from DEV
>>>>>>> dev
```

Meaning:

- HEAD → Current Branch
- ======= → Separator
- dev → Incoming Branch

---

# 9. Merge Conflict Resolution

```text
Merge Conflict
      │
      ▼
Open File
      │
      ▼
Choose Correct Content
      │
      ▼
Remove Conflict Markers
      │
      ▼
git add README.md
      │
      ▼
git commit
      │
      ▼
Merge Completed
```

---

# 10. Git Stash Workflow

```text
Working Directory
       │
Modified Files
       │
       ▼
   git stash
       │
       ▼
📦 Stash Storage
       │
Working Tree Clean
       │
Switch Branch
       │
Return Later
       │
git stash pop
       │
       ▼
Restore Changes
```

---

# 11. Git Stash vs Commit

```text
Unfinished Work

        │

 ┌──────┴────────┐
 │               │
 ▼               ▼

git commit    git stash

Permanent     Temporary

History       Hidden Storage
```

---

# 12. Git Tag

```text
Commit A

↓

Commit B

↓

🏷 v1.0

↓

Commit C

↓

Commit D

↓

🏷 v2.0
```

Tags permanently point to important commits.

---

# 13. Semantic Versioning

```text
v2.3.1

│ │ │

│ │ └──── Patch
│ │       Bug Fixes

│ └────── Minor
│          New Features

└──────── Major
           Breaking Changes
```

---

# 14. Git Workflow

```text
Create Branch
      │
      ▼
Write Code
      │
      ▼
git add
      │
      ▼
git commit
      │
      ▼
Merge Branch
      │
      ▼
Tag Release
      │
      ▼
Push to GitHub
```

---

# 15. Git Branch Lifecycle

```text
Create Branch
      │
      ▼
Develop Feature
      │
      ▼
Commit Changes
      │
      ▼
Merge to Main
      │
      ▼
Delete Branch
```

---

# 16. Feature Branch Workflow

```text
main

 │

 ├─────────────► feature-login

 │

 ├─────────────► feature-payment

 │

 ├─────────────► feature-dashboard

 │

 └─────────────► hotfix-production

↓

Merge

↓

main
```

---

# 17. Git Commands Flow

```text
git init
     │
     ▼
git status
     │
     ▼
git add .
     │
     ▼
git commit
     │
     ▼
git branch
     │
     ▼
git checkout
     │
     ▼
git merge
     │
     ▼
git stash
     │
     ▼
git tag
```

---

# 18. Complete Git Workflow

```text
                Working Directory
                       │
                  git add
                       │
                       ▼
                 Staging Area
                       │
                git commit
                       │
                       ▼
               Local Repository
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
      Branches     Git Tags     Git Stash
          │
          ▼
      Merge Changes
          │
          ▼
     Push to GitHub
```

---

# 19. Real DevOps Git Workflow

```text
Developer

     │

Create Feature Branch

     │

Write Code

     │

Commit Changes

     │

Push Branch

     │

Pull Request

     │

Code Review

     │

Merge to Main

     │

CI/CD Pipeline

     │

Production Deployment
```

---

# 20. Git Learning Roadmap

```text
Git Basics
     │
     ▼
Branching
     │
     ▼
Merge
     │
     ▼
Merge Conflict
     │
     ▼
Git Stash
     │
     ▼
Git Tags
     │
     ▼
GitHub
     │
     ▼
Pull Requests
     │
     ▼
CI/CD
```

---
