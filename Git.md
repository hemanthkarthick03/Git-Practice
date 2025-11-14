# Git Cheatsheet & Tricks

A comprehensive, Windows-friendly guide to essential Git commands, workflows, and advanced tricks with examples and gotchas.

> Tip: All commands below work in PowerShell. Lines starting with `#` are comments. For a few Unix one-liners (grep/xargs), use Git Bash or WSL.

## Table of Contents

- [Basic Commands](#basic-commands)
- [Repository Setup](#repository-setup)
- [Branching & Merging](#branching--merging)
- [Staging & Committing](#staging--committing)
- [Remote Operations](#remote-operations)
- [Viewing History](#viewing-history)
- [Undoing Changes](#undoing-changes)
- [Advanced Tricks](#advanced-tricks)
- [Git Workflows](#git-workflows)
- [Configuration](#configuration)
- [Aliases](#aliases)
- [SSH Configuration](#ssh-configuration)
- [Troubleshooting](#troubleshooting)
- [Performance Tips](#performance-tips)
- [Useful One-liners](#useful-one-liners)
- [Git Best Practices](#git-best-practices)

---

## Basic Commands

### Essential Daily Commands

```powershell
# Check status of working directory
git status

# Add files to staging area
git add <file>
git add .            # add everything in the current directory
git add -A           # add all changes (new, modified, deleted)
git add -u           # add modified and deleted, not new untracked files

# Commit changes
git commit -m "commit message"
git commit -am "message"  # add tracked changes and commit in one step

# Push to remote (push current branch to its upstream)
git push
git push origin main

# Pull from remote
git pull
git pull origin main

# Clone repository
git clone <url>
git clone <url> <directory>
```

### Quick Status Check

```powershell
git status -s       # short status format
git status -b       # show branch and tracking info
git status --ignored
```

---

## Repository Setup

### Initialize Repository

```powershell
git init                    # initialize new repository
git init -b main            # initialize with specific default branch name
git init --bare             # bare repo (for servers/remotes)
```

### First-Time Setup

```powershell
# User identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Default branch name for new repos
git config --global init.defaultBranch main

# Default editor
git config --global core.editor "code --wait"   # VS Code
# or
git config --global core.editor "vim"

# Line endings
git config --global core.autocrlf true    # Windows
# For Mac/Linux, prefer:
# git config --global core.autocrlf input

# Colorized output
git config --global color.ui auto

# Push behavior (push current branch to its upstream)
git config --global push.default simple
```

### Connect to Remote

```powershell
git remote add origin <url>
git remote -v
git remote set-url origin <new-url>
git remote remove origin
```

---

## Branching & Merging

### Branch Operations

```powershell
# List branches
git branch           # local branches
git branch -r        # remote branches
git branch -a        # all branches

# Create new branch
git branch <branch-name>
git checkout -b <branch-name>  # create and switch
git switch -c <branch-name>    # modern alternative

# Switch branches
git checkout <branch-name>
git switch <branch-name>       # modern alternative

# Delete branch
git branch -d <branch-name>    # safe delete (refuses if unmerged)
git branch -D <branch-name>    # force delete

# Rename branch
git branch -m <old-name> <new-name>
git branch -m <new-name>       # rename current branch
```

### Merging

```powershell
git merge <branch-name>            # merge branch into current branch
git merge --no-ff <branch-name>    # create a merge commit
git merge --squash <branch-name>   # combine all commits into one (then commit)
git merge --abort                  # abort merge if conflicts
```

### Rebasing

```powershell
git rebase <branch-name>       # rebase current branch onto another
git rebase -i HEAD~3           # interactive rebase last 3 commits
git rebase --continue          # after resolving conflicts
git rebase --abort

# Typical: rebase onto main and push safely
git rebase main
git push --force-with-lease    # safer than --force
```

> Note: Never rebase commits that others have already pulled unless coordinated; prefer merge for shared history.

---

## Staging & Committing

### Staging Files

```powershell
git add <file>             # add specific file
git add <directory>/       # add all files in a directory
git add -u                 # add modified/deleted only
git add -A                 # add new/modified/deleted
git add -p <file>          # interactively stage hunks

# Unstage (remove from index)
git reset HEAD <file>
git restore --staged <file>
```

### Committing

```powershell
git commit -m "commit message"
git commit -am "message"                         # add tracked changes and commit
git commit --amend                               # amend last commit (edit message or content)
git commit --amend -m "new message"              # amend just the message
git commit --date="2023-01-01 12:00:00" -m "msg" # set a specific date
git commit --allow-empty -m "trigger build"      # empty commit (e.g., trigger CI)

# Commit with title and body
git commit -m "Title" -m "Detailed description body"
```

### Commit Message Best Practices

- Format: `<type>(<scope>): <subject>`
- Use present tense and imperative mood
- Keep subject <= 72 chars; add details in the body

Examples:

```powershell
git commit -m "feat(auth): add user login functionality"
git commit -m "fix(api): resolve null pointer exception in user service"
git commit -m "docs(readme): update installation instructions"
git commit -m "refactor(utils): extract common validation logic"
```

---

## Remote Operations

### Working with Remotes

```powershell
# Fetch changes from remote
git fetch
git fetch origin
git fetch --all

# Pull (fetch + merge or rebase)
git pull
git pull origin main
git pull --rebase

# Push changes
git push
git push origin main
git push -u origin main        # set upstream and push
git push --all                 # push all branches
git push --tags                # push all tags
git push origin <tag-name>

# Force push (use with caution)
git push --force
git push --force-with-lease    # safer force push
```

### Tracking Branches

```powershell
# Set upstream tracking branch
git branch --set-upstream-to=origin/main main
git push -u origin main        # common way to set upstream on first push

# Create local branch tracking a remote branch
git checkout -b <local-branch> origin/<remote-branch>
git switch -c <local-branch> origin/<remote-branch>

# Delete remote branch
git push origin --delete <branch-name>
git push origin :<branch-name>   # alternative syntax
```

---

## Viewing History

### Log Commands

```powershell
git log                        # full log
git log --oneline              # one line per commit
git log --graph --oneline --all
git log -n 5                   # last 5 commits
git log --author="John Doe"
git log --since="2023-01-01" --until="2023-12-31"
git log <file>                 # commits affecting file
git log -p <file>              # with patches
git log --grep="bug fix"       # by message

# Beautiful log format
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

### Diff Commands

```powershell
git diff                       # unstaged changes
git diff --staged              # staged changes (alias: --cached)
git diff main..feature-branch  # compare branches
git diff <commit1> <commit2>   # compare commits
git diff <file>
git diff --word-diff           # word-level diff
git diff --name-only           # only file names
```

### Show Command

```powershell
git show <commit>              # show a commit
git show <commit>:<file>       # file at a commit
git show HEAD                  # last commit
git show --stat <commit>
```

---

## Undoing Changes

### Unstaging and Discarding

```powershell
# Unstage file
git reset HEAD <file>
git restore --staged <file>

# Discard changes in working directory (careful!)
git restore <file>
git restore .                  # discard all changes in tracked files

# Clean untracked files
git clean -n                   # dry run
git clean -f                   # remove untracked files
git clean -fd                  # remove untracked files and directories
```

### Reset Commands

```powershell
git reset --soft HEAD~1        # keep changes in staging
git reset --mixed HEAD~1       # default; keep changes in working tree
git reset HEAD~1
git reset --hard HEAD~1        # discard changes (irreversible)
git reset --hard <commit>
git reset HEAD <file>          # reset specific file (unstage)
```

### Revert Commands

```powershell
git revert <commit>            # create a new commit that reverts
git revert -m 1 <merge-commit> # revert a merge (choose parent)
git revert --no-commit <commit>
```

---

## Advanced Tricks

### Stashing

```powershell
git stash
git stash push -m "work in progress"
git stash list
git stash apply
git stash apply stash@{0}
git stash pop
git stash drop stash@{0}
git stash clear
git stash push <file1> <file2>
git stash -u                    # include untracked files
git stash branch <branch> stash@{0}
```

### Cherry-picking

```powershell
git cherry-pick <commit>
git cherry-pick <commit1> <commit2>
git cherry-pick <start-commit>..<end-commit>
git cherry-pick --no-commit <commit>
```

### Bisect (find the commit that introduced a bug)

```powershell
git bisect start
git bisect bad                      # current is bad
git bisect good <known-good-commit>
# Git checks out a midpoint; test and mark good/bad repeatedly
git bisect reset
```

### Worktrees

```powershell
git worktree list
git worktree add <path> <branch>
git worktree remove <path>
git worktree prune
```

### Submodules

```powershell
git submodule add <url> <path>
git submodule init
git submodule update
git clone --recursive <url>
git submodule update --remote
```

---

## Git Workflows

### Feature Branch Workflow

```powershell
# 1. Create feature branch
git checkout -b feature/new-feature

# 2. Work on feature
git add .
git commit -m "implement new feature"

# 3. Push feature branch
git push -u origin feature/new-feature

# 4. Open a pull request on GitHub/GitLab

# 5. After review, merge to main
git checkout main
git pull origin main
git merge feature/new-feature
git push origin main

# 6. Delete feature branch
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```

### Gitflow Workflow

> Requires the git-flow extension (not built into core Git). Install via your package manager or Git for Windows SDK, then:

```powershell
git flow init
git flow feature start <feature-name>
git flow feature finish <feature-name>
git flow release start <version>
git flow release finish <version>
git flow hotfix start <version>
git flow hotfix finish <version>
```

### Conventional Commits

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

```powershell
git commit -m "feat(auth): add OAuth2 integration"
git commit -m "fix(api): handle null response in user endpoint"
git commit -m "docs(readme): update installation guide"
```

---

## Configuration

```powershell
# User information
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Default editor
git config --global core.editor "code --wait"

# Default branch name
git config --global init.defaultBranch main

# Line endings
git config --global core.autocrlf true      # Windows
# git config --global core.autocrlf input   # Mac/Linux

# Color output
git config --global color.ui auto

# Default merge tool
git config --global merge.tool vimdiff

# Push behavior
git config --global push.default simple
```

---

## Aliases

### Useful Aliases

```powershell
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage "reset HEAD --"
git config --global alias.last "log -1 HEAD"
git config --global alias.visual "!gitk"
```

### Advanced Aliases

```powershell
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.adog "log --all --decorate --oneline --graph"
git config --global alias.pom "push origin main"
git config --global alias.poh "push origin HEAD"
```

---

## SSH Configuration

```powershell
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add SSH key to agent (Git Bash or Windows OpenSSH agent)
ssh-add $env:USERPROFILE\.ssh\id_ed25519

# Test SSH connection to GitHub
ssh -T git@github.com

# Use SSH remote URL for GitHub
git remote set-url origin git@github.com:username/repo.git
```

> If `ssh-add` isn't found in PowerShell, run from Git Bash or ensure the Windows OpenSSH Authentication Agent service is running.

---

## Troubleshooting

### Merge Conflicts

```powershell
# 1) Edit conflicted files and resolve markers (<<<<<<<, =======, >>>>>>>)
# 2) Stage resolved files
git add <resolved-file>
# 3) Complete merge
git commit
# Or abort merge
git merge --abort
```

### Detached HEAD

```powershell
git checkout -b <new-branch-name>   # create branch from current commit
git checkout main                   # or return to main branch
```

### Accidentally Committed to Wrong Branch

```powershell
# Move commits to correct branch
git checkout correct-branch
git cherry-pick <commit-hash>

# Remove commit from wrong branch (destructive)
git checkout wrong-branch
git reset --hard HEAD~1
```

### Large File Issues

Prefer using the community tool `git filter-repo` for history rewrites (faster, safer) or BFG.

```powershell
# filter-repo (install separately):
# git filter-repo --path <large-file> --invert-paths

# Legacy filter-branch (slow; not recommended)
git filter-branch --force --index-filter "git rm --cached --ignore-unmatch <large-file>" --prune-empty --tag-name-filter cat -- --all

# BFG Repo-Cleaner (Java)
java -jar bfg.jar --delete-files <large-file> .git
```

### Recovery Commands

```powershell
git reflog                                      # find lost commits
git checkout -b <branch-name> <commit-hash>     # recover deleted branch
git checkout <commit-hash> -- <file>            # recover deleted file
git fsck --lost-found                           # show all unreachable objects
```

---

## Git Hooks

> Hooks are scripts placed in `.git/hooks/`. Use Bash scripts on Windows via Git Bash, or set `core.hooksPath` to a directory you version-control.

### Pre-commit hook (run tests before commit)

```bash
#!/usr/bin/env bash
# .git/hooks/pre-commit
npm test
if [ $? -ne 0 ]; then
	echo "Tests failed, commit aborted"
	exit 1
fi
```

### Pre-push hook (run linting before push)

```bash
#!/usr/bin/env bash
# .git/hooks/pre-push
npm run lint
if [ $? -ne 0 ]; then
	echo "Linting failed, push aborted"
	exit 1
fi
```

### Commit message hook (enforce format)

```bash
#!/usr/bin/env bash
# .git/hooks/commit-msg
if ! grep -qE '^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+' "$1"; then
	echo "Invalid commit message format"
	exit 1
fi
```

---

## Performance Tips

### Speed Up Git

```powershell
git config --global core.preloadindex true
git config --global core.fsmonitor true     # requires fsmonitor support

# Partial and shallow clones
git clone --filter=blob:none <url>
git clone --depth 1 <url>

# Maintenance
git gc                 # garbage collection
git repack -ad         # repack objects
git prune              # remove unreachable objects
```

### Large Repository Handling

```powershell
# Modern sparse-checkout (Git >= 2.25)
git sparse-checkout init --cone
git sparse-checkout set src/

# Older approach
git config core.sparseCheckout true
"src/" | Out-File -Encoding ascii -FilePath .git/info/sparse-checkout
git read-tree -m -u HEAD

# Git LFS for large files
git lfs install
git lfs track "*.psd"
git add .gitattributes
```

---

## Useful One-liners

```powershell
git diff-tree --no-commit-id --name-only -r HEAD   # files changed in last commit
git shortlog -sn                                   # count commits by author
```

The following require a Unix-like shell (Git Bash/WSL):

```bash
git for-each-ref --format='%(committerdate) %09 %(refname)' | sort -k5n -k2M -k3n -k4n
git log -S "search_term" --oneline
git log main..HEAD --oneline
git format-patch -1 <commit>
git apply <patch-file>
git blame -w <file>
git log -L <start>,<end>:<file>
git merge --no-commit --no-ff <branch> && git merge --abort
git branch -r --merged
git branch --merged | grep -v "\*\|main\|master" | xargs -n 1 git branch -d
```

---

## Git Best Practices

### Commit Guidelines

- Write clear, descriptive commit messages
- Make atomic commits (one logical change per commit)
- Use present tense ("Add feature" not "Added feature")
- Keep commits small and focused
- Test before committing

### Branch Naming

Examples:

```
feature/user-authentication
bugfix/login-error
hotfix/security-patch
release/v1.2.0
```

### Repository Structure

```
.gitignore           # ignore patterns
README.md            # project documentation
CONTRIBUTING.md      # contribution guidelines
LICENSE              # license information
.github/             # GitHub specific files
	workflows/         # GitHub Actions workflows
	ISSUE_TEMPLATE/    # issue templates
PULL_REQUEST_TEMPLATE.md
```

### Security

- Never commit sensitive data (passwords, API keys)
- Use `.gitignore` for sensitive/local files
- Regularly scan repositories for secrets
- Use signed commits for important repositories

```powershell
git config --global commit.gpgsign true
git config --global user.signingkey <key-id>
```

---

Keep this cheatsheet handy for quick reference during development. If you need PDF/printable formats or team-specific conventions, consider exporting this file or adding a project-specific section.

