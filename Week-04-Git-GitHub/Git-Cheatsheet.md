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
