# Git Interview — Last-Minute Prep Guide

> **How to use this:** Skim the **Theory** for crisp definitions, read the **Commands** to see it in action, and rehearse the **Interview Q&A** out loud. Bold lines are the one-liners interviewers want to hear. Built for a 2–4 hour final revision.

---

## Table of Contents

1. [Git Fundamentals](#1-git-fundamentals)
2. [The Three Areas & Git Object Model](#2-the-three-areas--git-object-model)
3. [Core Workflow Commands](#3-core-workflow-commands)
4. [Branching & HEAD](#4-branching--head)
5. [Merge vs Rebase](#5-merge-vs-rebase)
6. [Reset vs Revert vs Checkout/Restore](#6-reset-vs-revert-vs-checkoutrestore)
7. [Stash](#7-stash)
8. [Cherry-Pick](#8-cherry-pick)
9. [Remotes: fetch, pull, push](#9-remotes-fetch-pull-push)
10. [Merge Conflicts](#10-merge-conflicts)
11. [Undoing & Recovering (reflog)](#11-undoing--recovering-reflog)
12. [Tags, Squash & Interactive Rebase](#12-tags-squash--interactive-rebase)
13. [.gitignore, diff, log & Inspection](#13-gitignore-diff-log--inspection)
14. [Git Workflows & Branching Strategies](#14-git-workflows--branching-strategies)
15. [Common Scenarios (with commands)](#15-common-scenarios)
16. [Tricky / Gotcha Questions](#16-tricky--gotcha-questions)
17. [Rapid-Fire One-Liners](#17-rapid-fire-one-liners)
18. [Last-Minute Checklist](#18-last-minute-checklist)

---

## 1. Git Fundamentals

### Theory

**Git** is a **distributed version control system (DVCS)** that tracks changes to files over time, enabling collaboration, history, and branching. Every developer has a **full local copy** of the repository (history included), so most operations are fast and work offline.

- **Version control system (VCS)** — tracks changes, enables collaboration and rollback.
- **Distributed (vs centralized like SVN)** — every clone is a complete repository with full history, not just a working copy. No single point of failure.
- **Git vs GitHub/GitLab:** **Git** is the tool/CLI; **GitHub/GitLab/Bitbucket** are **hosting platforms** that add collaboration features (pull requests, issues, CI/CD) on top of Git.
- **Snapshots, not diffs:** Git stores each commit as a **snapshot** of the whole project (unchanged files are referenced, not duplicated), unlike systems that store file-by-file deltas.

### Interview Q&A

**Q: What is Git and why use it?**
Git is a **distributed version control system** that tracks file changes, supports **branching/merging**, and lets teams collaborate with full history. Every clone is a **complete repo**, so it's fast, works offline, and has no single point of failure.

**Q: Git vs GitHub?**
**Git** is the version-control **tool** you run locally. **GitHub** (or GitLab/Bitbucket) is a **hosting service** for Git repositories that adds collaboration features — pull requests, code review, issues, and CI/CD. You can use Git without GitHub.

**Q: Centralized vs distributed VCS?**
In a **centralized** VCS (SVN), there's one central server and clients hold only a working copy. In a **distributed** VCS (Git), **every developer has the full repository with complete history**, enabling offline work, fast operations, and resilience.

**Q: Does Git store diffs or snapshots?**
**Snapshots.** Each commit records the state of the whole project; unchanged files are stored as references to the previous identical content (deduplicated), which makes branching and history operations efficient.

---

## 2. The Three Areas & Git Object Model

### Theory

Git has **three main states/areas** a file moves through:

1. **Working Directory** — your actual files on disk (modified, untracked).
2. **Staging Area (Index)** — a snapshot staged for the next commit (`git add` puts files here).
3. **Repository (.git)** — committed history (`git commit` saves the staged snapshot permanently).

(Plus the **remote** — the shared server copy.)

**Git object model (under the hood):**
- **Blob** — file contents.
- **Tree** — directory structure (maps names → blobs/trees).
- **Commit** — a snapshot pointer (tree + parent commit(s) + author + message).
- **Tag** — a named pointer to a commit.
Everything is identified by a **SHA-1/SHA-256 hash**.

### Commands

```bash
git status          # see working dir + staging state
git add file.txt    # working dir -> staging
git add .           # stage all changes
git commit -m "msg" # staging -> repository
git restore --staged file.txt   # unstage (staging -> working)
```

### Interview Q&A

**Q: Explain the three areas in Git.**
**Working directory** (your live files), **staging area/index** (changes marked for the next commit via `git add`), and the **repository** (committed history via `git commit`). The staging area lets you craft exactly what goes into a commit.

**Q: What is the staging area and why is it useful?**
A **buffer between your working directory and a commit**. It lets you **selectively choose** which changes to include in the next commit (e.g., stage one file, leave another), enabling clean, focused commits.

**Q: What's the difference between `git add` and `git commit`?**
`git add` moves changes from the **working directory to the staging area**. `git commit` permanently records the **staged snapshot** into the repository with a message.

**Q: What is a commit (internally)?**
A commit is a **snapshot** of the project — it points to a **tree** (the directory state), one or more **parent commits**, and metadata (author, message, timestamp). It's identified by a **hash**.

---

## 3. Core Workflow Commands

### Theory

The everyday loop: **edit → stage → commit → push** (and **pull** to sync).

### Commands

```bash
git init                    # create a new repo
git clone <url>             # copy a remote repo locally

git status                  # what's changed/staged
git add <file> | .          # stage changes
git commit -m "message"     # record staged changes
git commit -am "msg"        # stage tracked files + commit in one step

git log --oneline --graph   # view history compactly
git diff                    # working dir vs staging
git diff --staged           # staging vs last commit

git push origin main        # send commits to remote
git pull origin main        # fetch + merge remote changes
```

### Interview Q&A

**Q: `git init` vs `git clone`?**
`git init` **creates a new empty repository** in the current directory. `git clone` **copies an existing remote repository** (with full history) to your machine and sets up the `origin` remote.

**Q: What does `git commit -am` do?**
It **stages all modified tracked files and commits** them in one step. Note: it does **not** include **new/untracked** files — those still need `git add`.

**Q: How do you see what changed?**
`git status` shows which files are modified/staged/untracked. `git diff` shows the actual line changes (working vs staged); `git diff --staged` shows staged vs last commit. `git log` shows commit history.

---

## 4. Branching & HEAD

### Theory

A **branch** is just a **lightweight, movable pointer to a commit**. Creating a branch is cheap (no file copying). **HEAD** is a pointer to the **current branch/commit** you're working on.

- **`main`/`master`** — the default branch.
- **Feature branches** — isolate work; merge back when done.
- **Detached HEAD** — HEAD points directly to a commit, not a branch (e.g., after checking out a commit hash); new commits there can be lost if you switch away without a branch.

### Commands

```bash
git branch                      # list branches
git branch feature              # create a branch
git switch feature              # switch to it (modern)
git checkout feature            # switch (older syntax)
git switch -c feature           # create + switch
git checkout -b feature         # create + switch (older)

git branch -d feature           # delete (merged)
git branch -D feature           # force delete (unmerged)
git branch -m old new           # rename
```

### Interview Q&A

**Q: What is a branch in Git?**
A **movable pointer to a commit**. Branches are lightweight (just a reference), so creating/switching is instant. They let you **isolate work** (features/fixes) and merge it back without disturbing `main`.

**Q: What is HEAD?**
**HEAD is a pointer to the current commit/branch** you have checked out — essentially "where you are." It usually points to the tip of the current branch.

**Q: What is a detached HEAD state?**
When **HEAD points directly to a commit** instead of a branch (e.g., after `git checkout <hash>`). You can look around and commit, but those commits **aren't on any branch** and can be lost — create a branch (`git switch -c`) to keep them.

**Q: `git switch` vs `git checkout`?**
`git switch` (and `git restore`) are **newer, clearer commands** — `switch` changes branches, `restore` restores files. `git checkout` is the **older, overloaded** command that does both (and more), which is why the newer ones were introduced.

---

## 5. Merge vs Rebase

### Theory

Both **integrate changes** from one branch into another, but differently:

- **`git merge`** — combines branches by creating a **merge commit**, **preserving history** exactly (you can see where branches diverged and joined). Non-destructive. Can clutter history with merge commits.
- **`git rebase`** — **moves/replays** your commits **on top of** another branch's tip, producing a **clean, linear history** with no merge commits. **Rewrites history** (new commit hashes).

**Fast-forward merge** — if the target branch hasn't diverged, Git just moves the pointer forward (no merge commit). Use `--no-ff` to force a merge commit.

**The golden rule of rebasing:** **never rebase commits that have been pushed/shared**, because rewriting shared history breaks everyone else's repo. Rebase **local/private** branches only.

### Commands

```bash
git checkout feature
git merge main              # merge main into feature (merge commit if diverged)

git checkout feature
git rebase main             # replay feature's commits on top of main

git rebase --continue       # after resolving conflicts mid-rebase
git rebase --abort          # bail out
```

### Interview Q&A

**Q: Merge vs rebase — what's the difference?**
**Merge** integrates branches with a **merge commit**, preserving the true history (shows divergence/joining). **Rebase** **replays your commits onto the target branch's tip**, creating a **clean linear history** but **rewriting commit hashes**. Merge = non-destructive/honest history; rebase = clean/linear history.

**Q: When would you use rebase over merge?**
Use **rebase** to keep a **clean, linear history** — e.g., updating a **local feature branch** with the latest `main` before opening a PR. Use **merge** when integrating into shared branches or when you want to **preserve the actual branch topology**.

**Q: What's the golden rule of rebase?**
**Never rebase commits that are already pushed/shared.** Rewriting public history changes hashes and forces painful recovery for collaborators. Only rebase **private/local** branches.

**Q: What is a fast-forward merge?**
When the target branch has **no new commits** since the feature branched, Git simply **moves the branch pointer forward** to the feature's tip — no merge commit. Use `git merge --no-ff` to force a merge commit for traceability.

---

## 6. Reset vs Revert vs Checkout/Restore

### Theory

These all "undo," but at different scopes — a frequent interview topic.

- **`git revert <commit>`** — creates a **new commit that undoes** a previous commit. **Safe for shared/pushed history** (doesn't rewrite). Use on `main`.
- **`git reset`** — **moves the branch pointer** to an earlier commit, **rewriting history**. Three modes:
  - `--soft` — move HEAD; keep changes **staged**.
  - `--mixed` (default) — move HEAD; keep changes in **working directory** (unstaged).
  - `--hard` — move HEAD and **discard all changes** (dangerous).
- **`git restore` / `git checkout -- <file>`** — discard changes in the **working directory** for specific files (restore from last commit/staging).

### Commands

```bash
git revert <hash>            # safe undo on shared branches (new commit)

git reset --soft HEAD~1      # undo last commit, keep changes staged
git reset --mixed HEAD~1     # undo last commit, keep changes unstaged (default)
git reset --hard HEAD~1      # undo last commit AND discard changes (careful!)

git restore file.txt         # discard working-dir changes to a file
git restore --staged file.txt# unstage a file
```

### Interview Q&A

**Q: Reset vs revert — when do you use each?**
**`revert`** creates a **new commit that undoes** a change — **safe on shared/pushed branches** because it doesn't rewrite history. **`reset`** **moves the branch pointer backward and rewrites history** — fine for **local, unpushed** work, dangerous on shared branches. Rule: **revert on public, reset on private**.

**Q: Explain `git reset` soft vs mixed vs hard.**
**`--soft`** moves HEAD back but **keeps changes staged**. **`--mixed`** (default) keeps changes in the **working directory but unstaged**. **`--hard`** moves HEAD and **discards all changes** — irreversible for uncommitted work.

**Q: How do you undo a commit that's already pushed?**
Use **`git revert <hash>`** — it adds a new commit that reverses the changes while **preserving history**, so collaborators aren't disrupted. Avoid `reset --hard` + force-push on shared branches.

**Q: How do you discard local changes to a file?**
`git restore <file>` (or older `git checkout -- <file>`) to discard working-directory changes; `git restore --staged <file>` to unstage.

---

## 7. Stash

### Theory

**`git stash`** temporarily **shelves uncommitted changes** (working dir + staged) so you get a clean working tree — useful when you need to **switch branches or pull** but aren't ready to commit. You reapply them later.

### Commands

```bash
git stash                 # shelve changes (clean working tree)
git stash push -m "wip"   # stash with a label
git stash list            # see stashes
git stash pop             # reapply most recent stash and remove it
git stash apply           # reapply but keep the stash
git stash drop            # delete a stash
git stash -u              # include untracked files
```

### Interview Q&A

**Q: What is `git stash` and when do you use it?**
It **temporarily saves uncommitted changes** and reverts your working directory to clean, so you can **switch contexts** (e.g., fix an urgent bug on `main`) without committing half-done work. Later, `git stash pop` reapplies it.

**Q: `git stash pop` vs `git stash apply`?**
**`pop`** reapplies the latest stash **and removes it** from the stash list. **`apply`** reapplies it but **keeps it** in the list (useful if you want to apply the same stash to multiple branches).

**Q: Scenario — you're mid-feature and an urgent prod bug comes in. What do you do?**
`git stash` my work → `git switch main` → fix and commit the bug → switch back to the feature branch → `git stash pop` to resume exactly where I left off.

---

## 8. Cherry-Pick

### Theory

**`git cherry-pick <commit>`** applies a **specific commit** from one branch onto your current branch — copying just that change (a new commit with a new hash). Useful for **hotfixes**, pulling **one feature commit** without merging the whole branch, or **moving a commit made on the wrong branch**.

### Commands

```bash
git checkout main
git cherry-pick <hash>          # apply that commit onto main
git cherry-pick <hash1> <hash2> # multiple commits
git cherry-pick A..B            # a range
```

### Interview Q&A

**Q: What is `git cherry-pick` and when do you use it?**
It **applies a single specific commit** from another branch onto the current one. Common uses: **backporting a hotfix** to a release branch, taking **one needed commit** without merging an entire feature, or **moving a commit committed to the wrong branch**.

**Q: Cherry-pick vs merge?**
**Merge** brings in **all commits** of a branch (with history). **Cherry-pick** copies a **single (or few) selected commits**. Cherry-pick is surgical; merge is wholesale.

**Q: Cherry-pick vs revert?**
**Cherry-pick** *applies* a commit's changes elsewhere; **revert** *undoes* a commit by creating an inverse commit. Cherry-pick adds, revert subtracts.

---

## 9. Remotes: fetch, pull, push

### Theory

A **remote** is a shared version of the repo (e.g., on GitHub), named **`origin`** by default.

- **`git fetch`** — downloads remote commits/refs **but does not merge** them into your working branch (safe; updates your remote-tracking branches).
- **`git pull`** — **`fetch` + `merge`** (or `fetch` + `rebase` with `--rebase`); updates your current branch with remote changes.
- **`git push`** — uploads your local commits to the remote.

### Commands

```bash
git remote -v                       # list remotes
git remote add origin <url>
git fetch origin                    # download, don't merge
git pull origin main                # fetch + merge
git pull --rebase origin main       # fetch + rebase (linear)
git push origin main
git push -u origin feature          # push + set upstream tracking
git push --force-with-lease         # safer force push
```

### Interview Q&A

**Q: `git fetch` vs `git pull`?**
**`fetch`** downloads remote changes into your **remote-tracking branches** but **doesn't modify your working branch** — safe to inspect first. **`pull`** is **`fetch` + `merge`** (or rebase) — it downloads **and integrates** into your current branch in one step.

**Q: What is `origin`?**
The **default name for the remote repository** you cloned from. It's just a conventional alias for the remote URL.

**Q: `git push --force` vs `--force-with-lease`?**
`--force` **overwrites the remote unconditionally** (dangerous — can erase others' commits). **`--force-with-lease`** force-pushes **only if no one else has pushed** since your last fetch — a safer way to update a rewritten branch.

**Q: What does `-u`/`--set-upstream` do?**
It **links your local branch to a remote branch**, so future `git push`/`git pull` work without specifying the remote and branch.

---

## 10. Merge Conflicts

### Theory

A **merge conflict** happens when two branches change the **same lines** of a file (or one edits a file the other deleted) and Git can't auto-merge. Git marks the conflict with `<<<<<<<`, `=======`, `>>>>>>>` markers; you **manually resolve**, then stage and commit.

### Commands

```bash
git merge feature
# ... conflict ...
# Edit files to resolve the <<<< ==== >>>> markers
git add <resolved-file>
git commit                  # completes the merge
# or:
git merge --abort           # cancel the merge entirely

git diff                    # see conflicts
git checkout --theirs file  # take their version
git checkout --ours file    # take our version
```

### Interview Q&A

**Q: What causes a merge conflict and how do you resolve it?**
A conflict occurs when **both branches modify the same lines** (or conflicting file operations) so Git can't merge automatically. To resolve: open the marked files, **manually edit** to the desired result (removing the `<<<<`/`====`/`>>>>` markers), then `git add` the files and `git commit` to complete the merge. `git merge --abort` cancels it.

**Q: How do you avoid conflicts?**
Pull/rebase **frequently** to stay current, keep branches **short-lived and small**, **communicate** on shared files, and structure code to minimize overlapping edits.

**Q: What do `--ours` and `--theirs` mean?**
During a conflict, **`--ours`** keeps the **current branch's** version of a file and **`--theirs`** takes the **incoming branch's** version — shortcuts when you want one side wholesale. (Note: meanings flip during a rebase.)

---

## 11. Undoing & Recovering (reflog)

### Theory

**`git reflog`** records **every change to HEAD** (commits, checkouts, resets, rebases) — even "lost" commits. It's the **safety net**: you can recover commits after a bad `reset --hard`, rebase, or branch deletion by finding them in the reflog.

### Commands

```bash
git reflog                          # history of HEAD movements
git reset --hard HEAD@{2}           # jump back to a reflog entry
git checkout -b recovered <hash>    # rescue a lost commit into a branch

git commit --amend                  # fix the last commit (message/content)
```

### Interview Q&A

**Q: You did `git reset --hard` and lost commits. How do you recover them?**
Use **`git reflog`** — it logs every HEAD position, including the "lost" commit. Find the commit's hash/`HEAD@{n}` entry and recover it with `git reset --hard <hash>` or `git checkout -b rescue <hash>`. Commits aren't truly gone until garbage collection.

**Q: What is `git commit --amend`?**
It **modifies the most recent commit** — to fix the message or add forgotten changes. It **rewrites that commit** (new hash), so don't amend commits already pushed/shared.

**Q: How do you change a commit message that hasn't been pushed?**
`git commit --amend -m "new message"` for the last commit; for older commits use **interactive rebase** (`git rebase -i`) and mark them `reword`.

---

## 12. Tags, Squash & Interactive Rebase

### Theory

- **Tag** — a named pointer to a commit, used for **releases/versions**. **Annotated tags** (`-a`) store metadata (recommended for releases); **lightweight tags** are just a name.
- **Squash** — combine **multiple commits into one** for a clean history (e.g., squash a feature's WIP commits before merging). Done via interactive rebase or a squash merge.
- **Interactive rebase (`git rebase -i`)** — **edit, reorder, squash, drop, or reword** commits in a range.

### Commands

```bash
git tag v1.0                         # lightweight tag
git tag -a v1.0 -m "Release 1.0"     # annotated tag
git push origin v1.0                 # push a tag (not pushed by default)

git rebase -i HEAD~3                 # interactively edit last 3 commits
# in the editor: pick / squash / reword / drop / reorder

git merge --squash feature           # squash all feature commits into one staged change
```

### Interview Q&A

**Q: What is a tag and how does it differ from a branch?**
A **tag** is a **fixed pointer to a specific commit**, typically marking a **release** (e.g., `v1.0`). Unlike a **branch**, it **doesn't move** as new commits are made — it permanently marks that point.

**Q: Annotated vs lightweight tag?**
A **lightweight** tag is just a name pointing to a commit. An **annotated** tag (`-a`) is a **full object** storing tagger, date, message (and can be signed) — **recommended for releases**.

**Q: What is squashing and why do it?**
**Combining multiple commits into one** to produce a **clean, readable history** — e.g., collapsing many "WIP"/"fix typo" commits into a single meaningful commit before merging a feature. Done with `rebase -i` (mark commits `squash`) or a squash merge.

**Q: What can interactive rebase do?**
`git rebase -i` lets you **reword, reorder, squash, fixup, edit, or drop** commits in a range — a powerful tool for cleaning up local history before sharing.

---

## 13. .gitignore, diff, log & Inspection

### Theory

- **`.gitignore`** — lists patterns of files Git should **not track** (build artifacts, secrets, `node_modules`, logs). **Already-tracked files aren't ignored** — you must `git rm --cached` them first.
- **`git log`** — view history (with many formatting options).
- **`git diff`** — show changes between states.
- **`git blame`** — show who last changed each line.
- **`git show <commit>`** — view a commit's changes.

### Commands

```bash
git log --oneline --graph --all     # compact visual history
git log -p file.txt                 # history of a file with diffs
git diff HEAD~1 HEAD                 # changes between two commits
git blame file.txt                  # line-by-line authorship
git show <hash>                     # what a commit changed

git rm --cached secret.env          # stop tracking a file (keep on disk)
```

### Interview Q&A

**Q: What is `.gitignore` and a common gotcha?**
A file listing **patterns Git should not track** (artifacts, secrets, dependencies). **Gotcha:** it only affects **untracked** files — if a file is **already tracked**, adding it to `.gitignore` won't ignore it; you must `git rm --cached <file>` first.

**Q: How do you find who changed a specific line?**
`git blame <file>` shows the **commit and author** responsible for each line — useful for understanding why code changed.

**Q: How do you see what a specific commit changed?**
`git show <hash>` displays the commit's metadata and **diff**. `git log -p` shows diffs across history.

---

## 14. Git Workflows & Branching Strategies

### Theory

Common team workflows:

- **Feature Branch Workflow** — each feature in its own branch off `main`, merged via **pull request** after review. Simple, common.
- **Gitflow** — structured branches: `main` (production), `develop` (integration), plus `feature/`, `release/`, and `hotfix/` branches. Heavier; good for scheduled releases.
- **GitHub Flow** — lightweight: branch off `main`, open a PR, review, merge, deploy. Great for continuous deployment.
- **Forking Workflow** — each developer has their **own server-side fork**; contributions via PRs from forks. Common in open source.
- **Trunk-Based Development** — everyone commits to `main`/trunk frequently behind feature flags; minimal long-lived branches. Favors CI/CD.

### Interview Q&A

**Q: Describe a Git workflow you've used.**
"We used a **feature-branch workflow with pull requests**: branch off `main`, develop, push, open a **PR** for code review and CI checks, then merge (often **squash merge**) into `main`, which triggers deployment. Short-lived branches kept merges clean and conflicts rare."

**Q: Gitflow vs GitHub Flow vs trunk-based?**
**Gitflow** uses multiple long-lived branches (`main`, `develop`, release/hotfix) — structured, suited to **scheduled releases**. **GitHub Flow** is lightweight (branch → PR → merge → deploy) — suited to **continuous deployment**. **Trunk-based** has everyone commit to `main` frequently behind **feature flags** — optimized for **CI/CD and fast integration**.

**Q: What is a pull/merge request?**
A **request to merge one branch into another**, used for **code review, discussion, and automated checks (CI)** before integrating. It's the collaboration gate in most team workflows.

---

## 15. Common Scenarios

Practical "how do I…" questions with the commands.

**Undo the last commit but keep the changes:**
```bash
git reset --soft HEAD~1     # keeps changes staged
```

**Undo a pushed commit safely:**
```bash
git revert <hash>           # new commit reversing it
```

**Move an uncommitted change to a clean state to switch branches:**
```bash
git stash && git switch other-branch   # ... later: git stash pop
```

**Bring one commit from another branch:**
```bash
git cherry-pick <hash>
```

**Update your feature branch with latest main, keeping linear history:**
```bash
git switch feature && git rebase main
```

**Fix the last commit message:**
```bash
git commit --amend -m "better message"   # only if not pushed
```

**Recover a commit after a bad reset:**
```bash
git reflog                  # find the hash
git reset --hard <hash>
```

**Discard ALL local changes (careful):**
```bash
git reset --hard HEAD
git clean -fd               # also remove untracked files/dirs
```

**See commits in feature not in main:**
```bash
git log main..feature
```

**Rename the current branch:**
```bash
git branch -m new-name
```

### Interview Q&A

**Q: You committed to the wrong branch. How do you fix it?**
If not pushed: note the commit hash, switch to the correct branch, `git cherry-pick <hash>`, then go back and `git reset --hard HEAD~1` to remove it from the wrong branch. (Or use `git reset --soft HEAD~1` then move the staged changes.)

**Q: How do you completely remove untracked files?**
`git clean -fd` (force, directories). Use `git clean -n` first to **preview** what would be deleted.

---

## 16. Tricky / Gotcha Questions

**1. `reset` rewrites history; `revert` doesn't.** Use **revert on shared/pushed** branches, reset only on local.

**2. Never rebase pushed/shared commits** — rewriting public history breaks collaborators.

**3. `.gitignore` doesn't untrack already-tracked files** — `git rm --cached` first.

**4. `fetch` is safe; `pull` merges** — pull can cause unexpected merges/conflicts.

**5. `reset --hard` discards uncommitted work** — but committed work is recoverable via `reflog`.

**6. `commit --amend` rewrites the last commit** (new hash) — don't amend pushed commits.

**7. Fast-forward merge creates no merge commit** — use `--no-ff` if you want one for traceability.

**8. Detached HEAD commits can be lost** — create a branch to keep them.

**9. `git pull --rebase`** avoids unnecessary merge commits when syncing.

**10. Tags aren't pushed by default** — `git push origin <tag>` or `--tags`.

**11. Deleting a branch (`-d`) is blocked if unmerged** — `-D` forces it (you can lose commits).

**12. `--force-with-lease` is safer than `--force`** — it won't clobber others' new commits.

---

## 17. Rapid-Fire One-Liners

- **Git** = distributed VCS; every clone has full history.
- **Git** = tool; **GitHub/GitLab** = hosting + collaboration.
- **Three areas:** working directory → staging (index) → repository.
- **`add`** stages; **`commit`** records; **`push`** uploads; **`pull`** = fetch + merge.
- **Branch** = movable pointer to a commit; **HEAD** = where you are.
- **Merge** = preserves history (merge commit); **rebase** = linear history (rewrites).
- **Golden rule:** never rebase shared/pushed commits.
- **reset** = rewrite history (local); **revert** = safe undo (shared).
- **reset --soft** (staged) / **--mixed** (unstaged) / **--hard** (discard).
- **stash** = shelve uncommitted work; **pop** reapplies + removes.
- **cherry-pick** = apply one specific commit elsewhere.
- **fetch** = download only; **pull** = download + integrate.
- **origin** = default remote name.
- **Merge conflict** = same lines changed; resolve, `add`, `commit`.
- **reflog** = HEAD history; recover lost commits.
- **commit --amend** = fix last commit (rewrites hash).
- **Tag** = fixed pointer (release); annotated vs lightweight.
- **Squash** = combine commits for clean history; `rebase -i`.
- **`.gitignore`** = untracked-only; `rm --cached` for tracked.
- **`--force-with-lease`** > `--force`.
- **Workflows:** feature-branch, Gitflow, GitHub Flow, forking, trunk-based.

---

## 18. Last-Minute Checklist

The hour before:

- [ ] Git vs GitHub; distributed vs centralized; snapshots not diffs.
- [ ] Three areas: working dir, staging, repository; what `add`/`commit` do.
- [ ] Branch & HEAD; detached HEAD; `switch` vs `checkout`.
- [ ] **Merge vs rebase** + the golden rule (the #1 question).
- [ ] **Reset (soft/mixed/hard) vs revert** + when each is safe.
- [ ] **Stash** (pop vs apply) and the urgent-bug scenario.
- [ ] **Cherry-pick** vs merge vs revert.
- [ ] **fetch vs pull**; force vs force-with-lease; upstream tracking.
- [ ] **Merge conflicts**: how they arise and how to resolve.
- [ ] **reflog** recovery; `commit --amend`.
- [ ] Tags (annotated vs lightweight); squash; interactive rebase.
- [ ] `.gitignore` gotcha (already-tracked files).
- [ ] Branching strategies (feature-branch, Gitflow, GitHub Flow, trunk-based).
- [ ] Be ready for "how do I undo/move/recover…" scenario commands (§15).

**Interview tips:** The most-asked Git questions are **merge vs rebase**, **reset vs revert**, and **how to undo/recover** — know these cold and *when each is safe on shared vs local branches*. Mention **pull requests, code review, and short-lived feature branches** when asked about workflow — it shows team maturity. If you've used Git in CI/CD (you have, per your background), frame answers around a real **feature-branch + PR + automated-deploy** workflow.

---

*Good luck — read it top-to-bottom once, then drill merge-vs-rebase, reset-vs-revert, and the undo/recover scenarios in §15. Those decide Git interviews.*
