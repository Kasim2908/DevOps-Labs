# Advanced Git Cheatsheet

## Reset

| Command | Description |
|----------|-------------|
| git reset --soft HEAD~1 | Undo commit, keep staged changes |
| git reset HEAD~1 | Undo commit, keep working files |
| git reset --hard HEAD~1 | Undo commit and discard changes |

---

## Revert

```bash
git revert HEAD
```

Safely undo the latest commit.

---

## Cherry-pick

```bash
git cherry-pick <commit-id>
```

Copy one commit from another branch.

---

## Rebase

```bash
git rebase main
```

Move current branch commits onto another branch.

Interactive:

```bash
git rebase -i HEAD~3
```

---

## Reflog

```bash
git reflog
```

Recover lost commits.

Recover:

```bash
git reset --hard HEAD@{1}
```

---

## Detached HEAD

```bash
git checkout <commit-id>
```

Return to branch:

```bash
git checkout main
```

---

## Merge vs Rebase

| Merge | Rebase |
|--------|---------|
| Creates Merge Commit | No Merge Commit |
| Preserves History | Rewrites History |
| Good for Shared Branches | Good for Local Cleanup |

---

## Reset vs Revert

| Reset | Revert |
|--------|---------|
| Removes Commits | Creates Reverse Commit |
| Rewrites History | Preserves History |
| Local Development | Shared Repositories |
