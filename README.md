# bongo-devops-core

Git fundamentals assignment for the Bongodev DevOps course.

This repository is a practice ground. The files inside are mostly empty or hold a line of dummy text. What matters here is the commit history and the branch activity, because each task was about a specific Git workflow: ignoring secrets, working on branches, pushing to a remote, stashing, squashing, resolving conflicts, and recovering lost work.

Everything was done on Windows using Git Bash (MINGW64).

```
Author : asifaowadud
Remote : https://github.com/abuabddullah/bongo-devops-core
Branches: main, feature/system-optimization
```

---

## Quick overview

| Task | Topic              | Main commands                                    |
| ---- | ------------------ | ------------------------------------------------ |
| 1    | Repository setup   | `git init`, `git config`, `git commit`           |
| 2    | Ignoring secrets   | `.gitignore`, `git status`                       |
| 3    | Branch isolation   | `git checkout -b`, `git switch`                  |
| 4    | Atomic commits     | `git add <file>`, separate commits               |
| 5    | Remote and push    | `git branch -m`, `git remote add`, `git push -u` |
| 6    | History inspection | `git log -p`, `git blame`                        |
| 7    | Stashing work      | `git stash -u`, `git stash pop`                  |
| 8    | Amend and squash   | `git commit --amend`, `git merge --squash`       |
| 9    | Merge conflict     | `git merge`, manual resolve                      |
| 10   | Recovery           | `git reset --hard`, `git reflog`                 |

---

## Task 1: Repository setup

Created a new local repository and checked that my identity was set, so every commit gets the right author name and email.

```bash
git init
git config user.name          # asifaowadud
git config user.email         # asifaowadud@gmail.com
touch README.md
git add .
git commit -m "chore: initial repository setup"
```

The first commit included `README.md` and a `.gitignore` file. The `.gitignore` was prepared at this stage so that sensitive files would never be tracked from day one.

```
.env

bongo-devops-core
bongo-devops-core.*
.bongo-devops-core*
```

---

## Task 2: Keeping secrets out of Git

Created a `.env` file with a dummy password to simulate a real secret.

```bash
touch .env
echo "pass=0000" > .env
ls -la
git status
```

`ls -la` shows `.env` exists on disk, but `git status` reports:

```
nothing to commit, working tree clean
```

That confirms `.gitignore` is doing its job. Git does not even see the file, so there is no risk of pushing it by accident.

---

## Task 3: Branch isolation

Created a feature branch, committed a file there, then switched back to master to confirm the file does not leak into other branches.

```bash
git checkout -b feature/system-optimization
touch kernel_tuning.txt
git add kernel_tuning.txt
git commit -m "t3: kernel_tuning.txt created"

git switch master
ls -la
```

`kernel_tuning.txt` is not listed on master. Work on a branch stays on that branch until it is merged.

---

## Task 4: Atomic commits

Created two config files at once, but committed them separately. One change per commit keeps the history easy to read and easy to revert.

```bash
touch web_fix.conf db_fix.con

git add web_fix.conf
git commit -m "t4: only web_fix.conf tracked only"

git add db_fix.con
git commit -m "t4: only db_fix.conf tracked only"
```

Instead of `git add .`, each file was staged by name. If one fix turns out to be wrong later, it can be reverted without touching the other.

---

## Task 5: Renaming the branch and pushing to GitHub

Renamed `master` to `main` to match the GitHub default, connected the remote, and pushed both branches.

```bash
git branch -m main
git remote add origin https://github.com/abuabddullah/bongo-devops-core.git
git remote -v
```

The first plain `git push --force` failed because the branch had no upstream yet:

```
fatal: The current branch main has no upstream branch.
```

Fixed it by setting the upstream while pushing:

```bash
git push --set-upstream origin main --force
```

`--force` was needed because the GitHub repo already had a commit (`34240c0`) that did not exist locally. The remote history was replaced with the local one.

Then pushed the feature branch too:

```bash
git switch feature/system-optimization
git push --set-upstream origin feature/system-optimization --force
```

After this, both `main` and `feature/system-optimization` exist on GitHub and track their local branches.

---

## Task 6: Inspecting history

Used two commands to audit what changed and who changed it.

```bash
git log -p
```

This shows every commit along with the actual diff. For example, the initial commit shows each line added to `.gitignore`.

```bash
git blame .gitignore
```

Output:

```
^0c9e818 (asifaowadud 2026-09-19 21:25:59 +0600 1) .env
^0c9e818 (asifaowadud 2026-09-19 21:25:59 +0600 2)
^0c9e818 (asifaowadud 2026-09-19 21:25:59 +0600 3) bongo-devops-core
^0c9e818 (asifaowadud 2026-09-19 21:25:59 +0600 4) bongo-devops-core.*
^0c9e818 (asifaowadud 2026-09-19 21:25:59 +0600 5) .bongo-devops-core*
```

Every line points to the root commit `0c9e818` (the `^` marks a boundary/root commit), the author, and the exact time it was written.

---

## Task 7: Stashing unfinished work

Scenario: I was in the middle of writing `feature.py` (not ready to commit) when an urgent fix was needed on `db_fix.con`.

```
feature.py (half done, untracked)
        |
        v
   git stash -u      -> work saved aside, directory clean
        |
        v
  fix db_fix.con     -> commit the urgent change
        |
        v
   git stash pop     -> feature.py comes back
```

```bash
git stash -u
echo "test2" > db_fix.con
git add db_fix.con
git commit -m "task07: stashed and worked"
git stash pop
```

The `-u` flag matters here. A plain `git stash` only saves tracked files, and `feature.py` was untracked, so without `-u` it would have stayed in the directory.

After `git stash pop`, Git showed `feature.py` back as an untracked file and dropped the stash entry. The urgent fix went in as its own commit, separate from the unfinished feature.

---

## Task 8: Amend and squash merge

On the feature branch, made three small commits to `kernel_tuning.txt`. The third one was committed with the wrong message (`t8: commit2` again), so I fixed it with amend.

```bash
git switch feature/system-optimization

echo "test" > kernel_tuning.txt
git add kernel_tuning.txt
git commit -m "t8: commit1"

echo "test2" >> kernel_tuning.txt
git add kernel_tuning.txt
git commit -m "t8: commit2"

echo "test3" >> kernel_tuning.txt
git add kernel_tuning.txt
git commit -m "t8: commit2"

git commit --amend      # message changed to "t8: commit3"
```

Then merged the branch into main as one single commit instead of three:

```bash
git switch main
git merge --squash feature/system-optimization
git commit -m "t8: squashed into a single commit"
```

Result on main:

```
aa21cde (HEAD -> main) t8: squashed into a single commit
6f1b127 (origin/main, origin/HEAD) t7 done
79950b1 task07: stashed and worked
31fc4e0 task07: stashed and worked
3a2fc09 t4: only db_fix.conf tracked only
aada53e t4: only web_fix.conf tracked only
0c9e818  chore: initial repository setup
```

Main gets a clean history with one entry for the whole feature, while the feature branch still keeps the detailed commits.

---

## Task 9: Creating and resolving a merge conflict

Simulated two developers writing different content to the same new file on different branches.

On `main`:

```bash
echo "my name is asif" > optimization.txt
git add optimization.txt
git commit -m "t9: asif wrote on optimization.txt"
```

On `feature/system-optimization`:

```bash
git switch feature/system-optimization
echo "my name is shuvo" > optimization.txt
git add optimization.txt
git commit -m "t9: shuvo wrote on optimization.txt in branch feature/system-optimization"
```

Merging back into main:

```bash
git switch main
git merge feature/system-optimization
```

```
CONFLICT (add/add): Merge conflict in optimization.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Git marked the conflict inside the file:

```
<<<<<<< HEAD
my name is asif
=======
my name is shuvo
>>>>>>> feature/system-optimization
```

Opened it in `vi`, removed the markers, and kept both lines since both changes were valid:

```
my name is asif
my name is shuvo
```

Then finished the merge:

```bash
git add optimization.txt
git commit -m "t9: conflict resolved"
```

---

## Task 10: Recovering a "lost" commit with reflog

First, made a bad commit that overwrote `optimization.txt`:

```bash
echo "random" > optimization.txt
git add optimization.txt
git commit -m "t10L: destroyed the optimization.txt"
```

Then removed that commit completely with a hard reset:

```bash
git reset --hard HEAD~1
```

```
HEAD is now at ce56fa9 t9: conflict resolved
```

At this point the commit no longer shows in `git log`. It looks gone. But Git keeps a private record of every place HEAD has pointed to:

```bash
git reflog
```

```
ce56fa9 HEAD@{0}: reset: moving to HEAD~1
2445d62 HEAD@{1}: commit: t10L: destroyed the optimization.txt
ce56fa9 HEAD@{2}: commit (merge): t9: conflict resolved
...
```

The hash `2445d62` is still there. Resetting to it brought the commit back:

```bash
git reset --hard 2445d62
```

```
HEAD is now at 2445d62 t10L: destroyed the optimization.txt
```

The takeaway: `git reset --hard` is dangerous but not final. As long as the commit was made locally, `git reflog` can usually find it.

---

## Final repository structure

```
bongo-devops-core/
├── .env                 # ignored, never committed
├── .gitignore
├── README.md
├── db_fix.con
├── feature.py           # untracked, restored from stash
├── kernel_tuning.txt
├── optimization.txt
└── web_fix.conf
```

---

## What I practiced

| Concept                    | Why it matters                                              |
| -------------------------- | ----------------------------------------------------------- |
| `.gitignore`               | Keeps passwords and secrets out of the repository           |
| Branches                   | Work in isolation without breaking main                     |
| Atomic commits             | Clean history, easy to revert one change                    |
| Upstream tracking          | Lets `git push` and `git pull` work without extra arguments |
| `git log -p` / `git blame` | Find what changed, when, and by whom                        |
| `git stash -u`             | Park unfinished work, including untracked files             |
| `--amend` / `--squash`     | Fix mistakes and keep main history readable                 |
| Conflict resolution        | Handle two people editing the same file                     |
| `git reflog`               | Recover commits after a hard reset                          |
