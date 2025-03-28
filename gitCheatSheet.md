# Git Commands Cheat Sheet

## Commit Management
- `git cherry-pick <hash>`  
  Apply a specific commit to the current branch.

- `git revert <commit>`  
  Create a new commit that undoes the changes of a bad commit.

## Reset & Clean
- `git reset --soft [commit]`  
  Keep changes in staging area (index).

- `git reset --mixed [commit]` (default)  
  Unstage changes (moves to working directory).

- `git reset --hard [commit]`  
  **Dangerous**: Discard all changes permanently.

- `git reflog`  
  Recover from accidental resets by finding lost commits.

- `git clean`  
  Delete untracked files in working directory.  
  - `git clean -f` (force)  
  - `git clean -fd` (include directories)

## Branch & Checkout
- `git checkout <hash>`  
  Enter detached HEAD state (new commits will be lost unless branched).

- `git switch <branch>`  
  Switch branches (safer alternative to `checkout` for branches).

- `git checkout HEAD -- <filename>`  
  Discard changes to a specific file (working directory).

- `git restore <file>`  
  Better alternative to `checkout` for discarding changes:  
  - `git restore --staged <file>` (only unstage)  
  - `git restore --source <commit> <file>` (restore from a specific commit)

## Merge & Rebase
- `git merge --squash <branch>`  
  Combine all changes from `<branch>` into **one new commit** (does not merge branches).

- `git fetch --all`  
  Fetch updates from all remotes.

- `git pull`  
  Shortcut for `git fetch` + `git merge`.

- `git rebase <branch>`  
  Change the base of the current branch (rewrites commit history).

## Branch Management
- `git branch -vv`  
  List all branches and their upstream remotes.

- `git branch --unset-upstream`  
  Remove upstream tracking for a branch.

- `git branch <branch> -u origin/<branch>`  
  Set/change upstream for a branch.

## Stash
- `git stash push --staged`  
  Stash only staged changes.

## Submodules
- Git Submodules Tutorial:  
  [YouTube Guide](https://www.youtube.com/watch?v=qsTthZi23VE&t=1122s)

## Fork & Remote
- `git pull https://github.com/user/fork.git branch`  
  Pull from a fork without adding it as a remote.

- `git fork`  
  Use when you don’t have push access to a repo (creates a personal copy).

## Advanced
- `git submodule`  
  Manage nested repositories (see linked video above).