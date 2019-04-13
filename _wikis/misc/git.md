---
layout: article
title: Git
permalink: /wikis/misc/git
aside:
    toc: true
sidebar:
    nav: wikis
---


Git

## Git Basics

Create empty Git repo in specified directory. Run with no arguments to initialize the current directory as a git repository.
```bash
git init <directory>
```

Clone repo located at repo onto local machine. Original repo can be located n the local filesystem or on a remote machine via HTTP or SSH.
```bash
git clone <repo>
```

Stage all changes in directory for the next commit. Replace directory with a file to change a specific file.
```bash
git add <directory>
```

Commit the staged snapshot, but instead of launching a text editor, use message as the commit message.
```bash
git commit -m "<message>"
```

List which files are staged, unstaged, and untracked.
```bash
git status
```

Display the entire commit history using the default format.
For customization see additional options.
```bash
git log
```

Show unstaged changes between your index and working directory.
```bash
git diff
```


## Undoing Changes

Create new commit that undoes all of the changes made in <commit>, then apply it to the current branch.
```bash
git revert <commit>
```

Remove <file> from the staging area, but leave the working directory unchanged. This unstages a file without overwriting any changes.
```bash
git reset <file>
```

Shows which files would be removed from working directory. Use the -f flag in place of the -n flag to execute the clean.
```bash
git clean -n
```


## Git Branches

List all of the branches in your repo. Add a branch argument to create a new branch with the name branch.
```bash
git branch
```

Create and check out a new branch named branch. Drop the -b flag to checkout an existing branch.
```bash
git checkout -b <branch>
```

Merge branch into the current branch.
```bash
git merge <branch>
```


## Remote Repositories

Create a new connection to a remote repo. After adding a remote, you can use name as a shortcut for url in other commands.
```bash
git remote add <name> <url>
```

Fetches a specific branch, from the repo. Leave off branch to fetch all remote refs.
```bash
git fetch <remote> <branch>
```

Fetch the specified remote’s copy of current branch and immediately merge it into the local copy.
```bash
git pull <remote>
```

Push the branch to remote, along with necessary commits and objects. Creates named branch in the remote repo if it doesn’t exist.
```bash
git push <remote> <branch>
```


## git diff

Show difference between working directory and last commit.
```bash
git diff HEAD
```

Show difference between staged changes and last commit
```bash
git diff --cached
```


## git reset

Reset staging area to match most recent commit, but leave the working directory unchanged.
```bash
git reset
```

Reset staging area and working directory to match most recent commit and overwrites all changes in the working directory.
```bash
git reset --hard
```

Move the current branch tip backward to commit, reset the staging area to match, but leave the working directory alone.
```bash
git reset <commit>
```

Same as previous, but resets both the staging area & working directory to match. Deletes uncommitted changes, and all commits after commit.
```bash
git reset --hard <commit>
```


## git rebase

Interactively rebase current branch onto <base>. Launches editor to enter
commands for how each commit will be transferred to the new base.
```bash
git rebase -i <base>
```


## git pull

Fetch the remote’s copy of current branch and rebases it into the local
copy. Uses git rebase instead of merge to integrate the branches.
```bash
git pull --rebase <remote>
```


## git push

Forces the git push even if it results in a non-fast-forward merge. Do not use the --force flag unless you’re absolutely sure you know what you’re doing.
```bash
git push <remote> --force
```

Push all of your local branches to the specified remote.
```bash
git push <remote> --all
```

Tags aren’t automatically pushed when you push a branch or use the
--all flag. The --tags flag sends all of your local tags to the remote repo.
```bash
git push <remote> --tags
```

