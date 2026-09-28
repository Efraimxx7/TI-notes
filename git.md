# 🔀 Git — Essential Commands

> A practical reference of Git commands for daily work, team collaboration and fixing mistakes.

---

## 📋 Table of Contents

- [Setup](#setup)
- [Repository Basics](#repository-basics)
- [Branching](#branching)
- [Staging & Committing](#staging--committing)
- [Undoing Changes](#undoing-changes)
- [History & Investigation](#history--investigation)
- [Remotes](#remotes)
- [Tags & Releases](#tags--releases)
- [Signed Commits (GPG)](#signed-commits-gpg)
- [Removing Sensitive Files](#removing-sensitive-files)
- [CI/CD Usage](#cicd-usage)
- [Quick Reference](#quick-reference)
- [Recommended Tools](#recommended-tools)

---

## Setup

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global core.editor "nano"
git config --global credential.helper cache     # keep credentials in memory (not plaintext)
git config --global --list
```

---

## Repository Basics

```bash
git init
git clone git@github.com:user/repo.git          # SSH
git clone https://github.com/user/repo.git      # HTTPS
git status
git remote -v
git remote add origin git@github.com:user/repo.git
git remote remove origin
```

---

## Branching

```bash
git branch -a                          # list local and remote branches
git checkout -b feature/new-feature    # create and switch (or: git switch -c)
git checkout main                      # switch (or: git switch main)
git merge feature/new-feature
git rebase main                        # replay your commits on top of main
git branch -d feature/new-feature      # delete merged branch
git branch -D feature/new-feature      # force delete
```

---

## Staging & Committing

```bash
git add path/to/file
git add .                              # stage everything (check git status first)
git diff                               # unstaged changes
git diff --staged                      # review before committing
git commit -m "fix: correct config path"
git commit --amend -m "new message"    # fix last commit (before pushing)
git stash                              # save work temporarily
git stash list
git stash pop
```

---

## Undoing Changes

```bash
git restore path/to/file               # discard changes in a file
git restore --staged path/to/file      # unstage a file
git revert <commit>                    # new commit that undoes another (safe)
git reset --soft HEAD~1                # undo last commit, keep changes staged
git reset --hard <commit>              # go back and DISCARD changes (destructive)
git clean -n                           # preview untracked files to delete
git reflog                             # recover "lost" commits
```

**Merge conflicts**

```bash
git status                             # see conflicted files
# edit files, remove <<<<<<< ======= >>>>>>> markers
git add file.txt
git commit
git merge --abort                      # give up on the merge
```

---

## History & Investigation

```bash
git log
git log --oneline --graph --decorate --all
git log --all --grep="keyword"         # search commit messages
git log -p path/to/file                # history of one file
git log --author="name"
git log -S "text" --all --oneline      # commits that added/removed a string
git show <commit>
git diff <commit-A> <commit-B>
git blame path/to/file                 # who changed each line
git cherry-pick <commit>               # copy one commit to current branch

# Find the commit that introduced a bug
git bisect start
git bisect bad
git bisect good <commit>
git bisect reset
```

---

## Remotes

```bash
git fetch origin
git pull origin main
git push origin feature/new-feature
git push -u origin feature/new-feature   # set upstream
git push --force-with-lease              # safer force push (own branches only)
```

---

## Tags & Releases

```bash
git tag
git tag v1.0.0
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin --tags
git archive --format=tar.gz --output=release.tar.gz HEAD
```

---

## Signed Commits (GPG)

```bash
gpg --list-secret-keys --keyid-format=long
git config --global user.signingkey YOUR_KEY_ID
git config --global commit.gpgsign true
git commit -S -m "feat: add new module"
git log --show-signature -1
git tag -s v1.0.0 -m "Signed release"
git tag -v v1.0.0
```

---

## Removing Sensitive Files

If a password, key or `.env` file was committed by mistake:

```bash
git ls-files path/to/.env              # is it tracked?
git rm --cached path/to/.env           # stop tracking, keep local file

echo ".env" >> .gitignore
echo "*.pem" >> .gitignore
echo "*.key" >> .gitignore
git add .gitignore
git commit -m "chore: ignore sensitive files"

# Remove the file from ALL history
pip install git-filter-repo
git filter-repo --path path/to/secret.txt --invert-paths
git push origin --force --all
```

> **Important:** removing a file from history does not make the password safe. Always change/revoke the exposed credential.

---

## CI/CD Usage

```bash
ssh-keygen -t ed25519 -C "ci-deploy-key" -f ~/.ssh/deploy_key
GIT_SSH_COMMAND="ssh -i ~/.ssh/deploy_key" git clone git@github.com:user/repo.git
git clone --depth 1 git@github.com:user/repo.git     # shallow clone, faster pipelines
```

---

## Quick Reference

| Task | Command |
|---|---|
| See what changed | `git status` / `git diff` |
| Undo a file change | `git restore file` |
| Undo last commit (keep work) | `git reset --soft HEAD~1` |
| Safely undo a pushed commit | `git revert <commit>` |
| Find who changed a line | `git blame file` |
| Search history | `git log --all --grep="text"` |
| Save work temporarily | `git stash` / `git stash pop` |
| Recover lost work | `git reflog` |
| Prefer SSH over HTTPS | `git clone git@github.com:...` |

---

## Recommended Tools

| Tool | Purpose |
|---|---|
| [GitHub CLI (gh)](https://cli.github.com/) | Manage repos, PRs and issues from the terminal |
| [pre-commit](https://pre-commit.com/) | Run checks automatically before each commit |
| [gitleaks](https://github.com/gitleaks/gitleaks) | Detect passwords/keys accidentally committed |
| [git-filter-repo](https://github.com/newren/git-filter-repo) | Rewrite history (remove files) |
| [lazygit](https://github.com/jesseduffield/lazygit) | Terminal UI for Git |

---

<div align="center">

*Maintained as a personal reference for daily Git usage.*  
*Commands tested on Linux.*

</div>
