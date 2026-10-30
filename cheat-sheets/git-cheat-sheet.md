# Git cheat sheet

Git commands for everyday work and emergencies – searchable and copyable in one click. With warnings on commands that discard work for good.

## Everyday: status & commit

- Short status with branch and tracking info

  ```bash
  git status -sb
  ```
- Show unstaged changes

  ```bash
  git diff
  ```
- Show staged changes

  ```bash
  git diff --staged
  ```
- Stage changes hunk by hunk

  ```bash
  git add -p
  ```
- Create a commit with a message

  ```bash
  git commit -m "Fix login redirect"
  ```
- Add staged changes to the last commit (**careful**)

  ```bash
  git commit --amend --no-edit
  ```
- Create and switch to a new branch

  ```bash
  git switch -c feature/login
  ```

## Log & search

- Last 20 commits as a compact graph

  ```bash
  git log --oneline --graph --decorate -20
  ```
- One author's commits from the last 2 weeks

  ```bash
  git log --since='2 weeks ago' --author='Daniele'
  ```
- Find commits that added or removed a string

  ```bash
  git log -S 'password' --oneline
  ```
- Who last changed lines 10-30?

  ```bash
  git blame -L 10,30 file.py
  ```

## Stash

- Stash changes with a description

  ```bash
  git stash push -m "wip login"
  ```
- Also stash untracked files

  ```bash
  git stash push -u
  ```
- Apply a stash and keep it

  ```bash
  git stash apply stash@{0}
  ```
- Apply the newest stash and remove it

  ```bash
  git stash pop
  ```
- Discard one stash (**careful**)

  ```bash
  git stash drop stash@{0}
  ```

## Rebase & tidying history

- Replay the current branch onto main

  ```bash
  git rebase main
  ```
- Interactively reorder, squash or reword the last 4 commits (**careful**)

  ```bash
  git rebase -i HEAD~4
  ```
- Abort the rebase, back to the previous state

  ```bash
  git rebase --abort
  ```
- Merge a branch with a merge commit

  ```bash
  git merge --no-ff feature/login
  ```

## Undoing things

- Discard unstaged changes of a file (**careful**)

  ```bash
  git restore file.txt
  ```
- Unstage a file, keep the changes

  ```bash
  git restore --staged file.txt
  ```
- Undo a commit with a new inverse commit (safe on shared branches)

  ```bash
  git revert abc1234
  ```
- Undo the last commit, keep changes staged

  ```bash
  git reset --soft HEAD~1
  ```
- Throw away the last commit and its changes (**careful**)

  ```bash
  git reset --hard HEAD~1
  ```
- Permanently delete untracked files and folders (**careful**)

  ```bash
  git clean -fd
  ```

## Reflog & recovery

- Show the history of all HEAD movements

  ```bash
  git reflog
  ```
- Reset to an earlier reflog state (**careful**)

  ```bash
  git reset --hard HEAD@{2}
  ```
- Rescue a lost commit by hash as a branch

  ```bash
  git switch -c rescue abc1234
  ```
- Find dangling objects

  ```bash
  git fsck --lost-found
  ```
- Undo the last reset, merge or rebase (**careful**)

  ```bash
  git reset --hard ORIG_HEAD
  ```

## Remotes & bisect

- Fetch all remotes, prune deleted branches

  ```bash
  git fetch --all --prune
  ```
- Fetch and replay local commits on top

  ```bash
  git pull --rebase
  ```
- Force push that won't overwrite others' new commits (**careful**)

  ```bash
  git push --force-with-lease
  ```
- Start a binary search for the breaking commit

  ```bash
  git bisect start && git bisect bad && git bisect good v1.2
  ```
- Run bisect automatically with a test script

  ```bash
  git bisect run ./test.sh
  ```
- End bisect, return to the original branch

  ```bash
  git bisect reset
  ```

---
More detail, related guides and tools: **[daniele-michelin.com/en/cheat-sheets/git-cheat-sheet/](https://daniele-michelin.com/en/cheat-sheets/git-cheat-sheet/)**
