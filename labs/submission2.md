# Lab 2 — Version Control & Advanced Git

**Author:** Danis Sharafiev  

## Task 1 — Git Object Model Exploration

### 1.1 Sample Commit

```sh
echo "Test content" > test.txt
git add test.txt
git commit -m "Add test file"
```

### 1.2 Object Inspection

**Commit object** (`HEAD` → `091c246`):

```
tree dc239221d01b924e020a643dd8f0a01ba1296cb0
parent 2e9cdbe99e0c45815338f59d4edde6b14240ea5c
author Danis Sharafiev <harnemer@gmail.com> 1771002612 +0300
committer Danis Sharafiev <harnemer@gmail.com> 1771002612 +0300

Add test file
```

**Tree object** (`dc2392...`):

```
040000 tree 6a43605176f3ff1039f668fb82db4753edc0a560    .github
100644 blob 6e60bebec0724892a7c82c52183d0a7b467cb6bb    README.md
040000 tree a1061247fd38ef2a568735939f86af7b1000f83c    app
040000 tree 4d498781e84e8b9a3c6c69016e8b3868e0c5f9d2    labs
040000 tree d3fb3722b7a867a83efde73c57c49b5ab3e62c63    lectures
100644 blob 418a98ced2ac70b5bdee0be9732ecdaae7264515    test.txt
```

**Blob object** (`418a98...` — `test.txt`): contains the raw file content `"Test content"`.

### Analysis

| Object Type | Description |
|-------------|-------------|
| **Blob**    | Stores raw file content. No filename or metadata — just data. |
| **Tree**    | A directory listing: maps filenames/modes to blob or sub-tree hashes. |
| **Commit**  | Points to a tree (project snapshot), parent commit(s), author/committer info, and message. |

Git is a **content-addressable filesystem**: every piece of data is hashed (SHA-1) and stored as an immutable object. A commit references a tree, which recursively references blobs and sub-trees, forming a complete snapshot of the repository at that point in time.

---

## Task 2 — Reset and Reflog Recovery

### 2.1 Practice Commits

```sh
git switch -c git-reset-practice
echo "First commit"  > file.txt  && git add file.txt && git commit -m "First commit"   # 804c2a9
echo "Second commit" >> file.txt && git add file.txt && git commit -m "Second commit"  # 410c968
echo "Third commit"  >> file.txt && git add file.txt && git commit -m "Third commit"   # 7b49262
```

### 2.2 Reset Modes

| Command | HEAD | Index (staging) | Working Tree |
|---------|------|-----------------|--------------|
| `git reset --soft HEAD~1`  | ← moved back | **kept** | **kept** |
| `git reset --hard HEAD~1`  | ← moved back | **discarded** | **discarded** |

**After `--soft HEAD~1`:** HEAD moved from `7b49262` → `410c968`. Changes from "Third commit" stayed staged — ready to re-commit.

**After `--hard HEAD~1`:** HEAD moved from `410c968` → `804c2a9`. Both index and working tree were reset — "Second commit" and "Third commit" changes are gone from the working directory.

### 2.3 Recovery via Reflog

```
804c2a9 (HEAD -> git-reset-practice) HEAD@{0}: reset: moving to HEAD~1
410c968 HEAD@{1}: reset: moving to HEAD~1
7b49262 HEAD@{2}: commit: Third commit
410c968 HEAD@{3}: commit: Second commit
804c2a9 HEAD@{4}: commit: First commit
```

Recovery command:

```sh
git reset --hard 091c246   # restored HEAD to "Add test file"
```

**Key takeaway:** `git reflog` records every HEAD movement, so even after `--hard` reset, commits are recoverable as long as they haven't been garbage-collected (~90 days by default).

---

## Task 3 — Visualize Commit History

### Commands

```sh
git switch -c side-branch
echo "Branch commit" >> history.txt
git add history.txt && git commit -m "Side branch commit"
git switch -
git log --oneline --graph --all
```

### Graph Output

```
* 9ee1afa (side-branch) Side branch commit
* 091c246 (HEAD -> git-reset-practice, feature/lab2) Add test file
* 2e9cdbe (origin/feature/lab1, feature/lab1) docs: add screenshot
* 7a52d13 docs: add lab1 submission stub
* 69c05b0 docs: add commit signing summary
|/
* d6b6a03 (origin/main, origin/HEAD, main) Update lab2
* 87810a0 feat: remove old Exam Exemption Policy
* 1e1c32b feat: update structure
  ...
```

### Reflection

The `--graph` flag visualizes branches as parallel lines and merge points, making it immediately clear where work diverged and converged. This is invaluable for understanding project history and reviewing how features were integrated.

---

## Task 4 — Tagging Commits

### Commands

```sh
git tag v1.0.0
git push origin v1.0.0
```

### Result

| Tag | Commit | Description |
|-----|--------|-------------|
| `v1.0.0` | `091c246` | Lightweight tag on "Add test file" commit |

```
To https://github.com/DanisSharafiev/DevOps-Intro.git
 * [new tag]         v1.0.0 -> v1.0.0
```

### Why Tags Matter

Tags provide **stable, human-readable references** to specific commits. They are essential for:
- **Versioning** — marking release points (semantic versioning).
- **CI/CD** — triggering deployment pipelines on tag push events.
- **Release notes** — GitHub auto-generates release pages from tags.

---

## Task 5 — `git switch` vs `git checkout` vs `git restore`

### Commands and Outputs

**`git switch`** — branch operations:

```sh
git switch -c cmd-compare    # Switched to a new branch 'cmd-compare'
git switch -                 # Switched to branch 'git-reset-practice'
```

**`git checkout`** — legacy equivalent:

```sh
git checkout -b cmd-compare-2   # creates + switches (same result, overloaded command)
```

**`git restore`** — file operations:

```sh
echo "scratch" >> demo.txt
git add demo.txt
git restore demo.txt             # discards working tree changes (file stays staged)
git restore --staged demo.txt    # unstages file (moves back to untracked)
git restore --source=HEAD~1 demo.txt  # restores file content from a previous commit
```

`git status` after `git restore demo.txt`:

```
Changes to be committed:
        new file:   demo.txt
```

`git status` after `git restore --staged demo.txt`:

```
Untracked files:
        demo.txt
```

### Comparison

| Command | Scope | Use When |
|---------|-------|----------|
| `git switch` | **Branches only** | Creating/switching branches. Clear intent, no ambiguity. |
| `git checkout` | Branches **and** files | Legacy — avoid in new workflows; it's overloaded and confusing. |
| `git restore` | **Files only** | Discarding changes, unstaging files, or restoring from another commit. |

**Recommendation:** Use `git switch` + `git restore` instead of `git checkout`. The modern commands have a single responsibility each, reducing mistakes (e.g., accidentally overwriting files when you meant to switch branches).

---

## Task 6 — GitHub Community Engagement

### Why Starring Matters

Stars serve as bookmarks and social proof — they help you track useful projects and signal community trust, encouraging maintainers and attracting contributors.

### Why Following Matters

Following developers keeps you informed about their activity and discoveries, fostering professional networking, collaborative learning, and awareness of industry trends — skills essential beyond the classroom.
