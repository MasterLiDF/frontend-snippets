## I. Preparations: Basic Configuration & Repository Operations

### 1. Global User Information Configuration (Required for First Use)

This must be configured before committing code to associate identity information with commit records.

```bash

# Set username
git config --global user.name "Your Name"
# Set email
git config --global user.email "your.email@example.com"
# View all configurations to verify if they take effect
git config --list
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"
# Copy the public key
cat ~/.ssh/id_ed25519.pub | clip
```

✨ Note: Removing `--global` will only apply the configuration to the current repository, which can be used for multi-account development as needed.

### 2. Repository Initialization / Clone

#### Method 1: Create a new local repository

```bash

# Create an empty Git repository in the current directory
git init
# Associate the remote repository (for subsequent pushes)
git remote add origin Remote_Repository_URL(HTTPS/SSH)
```

#### Method 2: Clone an existing remote repository (Most commonly used)

```bash

# HTTPS method (No configuration needed, requires account password for each push)
git clone https://github.com/username/repo.git
# SSH method (Requires SSH key configuration, password-free login, Recommended)
git clone git@github.com:username/repo.git
# Clone and specify the local directory name
git clone Remote_Repository_URL my-repo
```

### 3. View Remote Repository Association Information

```bash

git remote -v
```

## II. Daily Development: Branch Operations (Git Core)

Development should be based on branches. The main branch (main/master) is only for releases, and feature development is completed on sub-branches.

### 1. View Branches

```bash

# View local branches (* marks the current branch)
git branch
# View all branches (local + remote)
git branch -a
```

### 2. Create & Switch Branch (Most commonly used)

#### Traditional Commands

```bash

# Create first, then switch
git branch Branch_Name(e.g. feature/payment)
git checkout Branch_Name
# Create and switch in one step (Recommended)
git checkout -b Branch_Name
```

#### Git 2.23+ New Commands (More semantic, replaces checkout)

```bash

# Switch to an existing branch
git switch Branch_Name
# Create and switch in one step (Recommended)
git switch -c Branch_Name
```

### 3. Pull Latest Code from Remote Branch

Pull first before development to avoid code conflicts.

```bash

# Pull code from the specified remote branch and automatically merge (Common for daily use)
git pull origin Branch_Name
# Only pull code without merging (Check differences first, then merge manually, for complex scenarios)
git fetch origin Branch_Name
```

## III. Development Completed: Staging & Committing Code

### 1. Check File Status (Frequently used during development)

Confirm modified, added, untracked files to avoid missing/wrong commits.

```bash

# Detailed output (Recommended, clearly shows file status)
git status
# Simplified output (One line per entry, quick view)
git status -s
```

### 2. Stage Modified Files

Move changes from the working directory to the staging area, preparing for subsequent commits.

```bash

# Stage specified single/multiple files
git add filename1.txt filename2.py
# Stage all modified/added files (Recommended, most commonly used in daily development)
git add .
# Stage all modifications (including deleted files, excluding new untracked files)
git add -u
# Interactive staging (Can stage partial content of files, for fine-grained commits)
git add -p
```

### 3. Commit Staged Code

Commit to the local repository, generating commit records. **Commit messages must follow conventions** (e.g. feat/fix/docs prefixes, for easy traceability).

```bash

# Commit and enter commit message manually (Requires entering the edit interface, save and exit)
git commit
# Commit directly with the commit message (Recommended, most commonly used)
git commit -m "feat: Add payment page feature"
# Amend the last commit (Modify commit message / supplement staged files, use only if not pushed to remote)
git commit --amend -m "fix: Fix payment page style issues"
```

### 4. View Commit History

Verify if the commit is successful, or trace historical changes.

```bash

# View all commit records (Reverse chronological order, detailed info)
git log
# Simplified output (One line per entry, shows commit ID and message, Recommended)
git log --oneline
# View commit history of a specified file
git log filename
# Visual branch merge graph (View commit history and merge relationships of all branches)
git log --graph --oneline --all
```

## IV. Feature Completed: Push to Remote & Merge Branches

### 1. Push Local Branch to Remote

```bash

# First push (Associate local and remote branch, no need for -u in subsequent pushes)
git push -u origin Branch_Name
# Non-first push (Push directly)
git push origin Branch_Name
```

### 2. Merge Branch to Main Branch (e.g. main)

It is recommended to use the `--no-ff` parameter to retain branch history, facilitating subsequent rollbacks, which complies with development conventions.

```bash

# 1. Switch to the main branch first
git switch main
# 2. Pull the latest code of the main branch (Avoid merge conflicts)
git pull origin main
# 3. Merge the feature branch into the main branch (Recommended --no-ff method)
git merge --no-ff Feature_Branch_Name(e.g. feature/payment)
# Perform merge operation, but will not automatically create a merge commit
git pull feature/a --no-commit
# 4. If conflicts occur, resolve them and execute the following command to continue merging
git add .
git merge --continue
# 5. After merging is complete, push the main branch to remote
git push origin main
```

### 3. Delete Branch (Cleanup after feature merge)

```bash

# Delete the local merged branch (Safe, will check if it's merged)
git branch -d Feature_Branch_Name
# Force delete the local unmerged branch (Caution: will lose unmerged code)
git branch -D Feature_Branch_Name
# Delete the remote branch (Sync cleanup after feature merge)
git push origin --delete Feature_Branch_Name
```

## V. Issue Handling: Undo/Rollback Operations (Emergency Essentials)

Sorted **from least to most impactful**, prioritize low-risk methods. **For code already pushed to remote, DO NOT use ** **`--hard`** ** forced rollback** (it will cause inconsistent team history).

### 1. Undo Working Directory Changes (Unstaged, Lightest)

Restore the file to the state of the last commit/stage, uncommitted changes will be overwritten.

```bash

# Traditional command
git checkout -- filename
# Git 2.23+ Recommended command (More semantic)
git restore filename
```

### 2. Undo Staging Area Changes (Staged, Uncommitted)

Move the staged file back to the working directory, retaining the modified content.

```bash

# Traditional command
git reset HEAD filename
# Git 2.23+ Recommended command
git restore --staged filename
```

### 3. Rollback Local Commits (Committed, Not Pushed to Remote)

#### Method 1: Retain Working Directory Changes (Only undo the commit, can re-commit)

```bash

# Rollback to the previous commit
git reset --soft HEAD^
# Rollback to a specified commit (Get commit_id via `git log --oneline`)
git reset --soft Commit_ID
```

#### Method 2: Complete Rollback (Delete all uncommitted changes in working directory, Use with caution)

```bash

# Rollback to the previous commit
git reset --hard HEAD^
# Rollback to a specified commit
git reset --hard Commit_ID
```

### 4. Rollback Remote Commits (Pushed to Remote, Recommended `git revert`, Risk-free)

Add a **reverse commit** to cancel the changes of the target commit, retain the original commit history, team-collaboration friendly.

```bash

# Revert a single normal commit
git revert Commit_ID
# Revert a merge commit with --no-ff (-m 1 means retain main branch code, discard feature branch code)
git revert -m 1 Merge_Commit_ID
```

✨ Note: For multiple commits to revert, execute **from back to front** to avoid code conflicts.

## VI. Version Management: Tag Operations

Used to mark version release nodes (e.g. v1.0.0), for easy version traceability and rollback, operations need to be synced to remote.

### 1. Create Tag (Associate with specified commit ID, Recommended with annotation)

```bash

# 1. View commit ID (Confirm the commit corresponding to the version)
git log --oneline
# 2. Create an annotated tag (-a=tag name, -m=tag annotation, Recommended)
git tag -a Tag_Name(e.g. v1.0.1) Commit_ID -m "Version 1.0.1: Optimize payment page style"
# 3. Push the tag to remote
git push origin Tag_Name
```

### 2. Delete Tag

```bash

# 1. Delete the local tag
git tag -d Tag_Name
# 2. Delete the remote tag
git push origin --delete Tag_Name
```

### 3. Rollback Version by Tag (Use with caution in production environment!)

After rollback, you need to force push, **only applicable for emergency situations**, need to communicate with the team in advance.

```bash

# 1. Switch to the target branch (e.g. main)
git switch main
# 2. Reset the branch to the commit corresponding to the tag (--hard overwrites working directory and staging area)
git reset --hard Tag_Name
# 3. Force push to remote (DO NOT use arbitrarily in production environment!)
git push -f origin main
```

## VII. Core Operation Cheat Sheet (Frequently Used in Daily Development)

### 1. Basic Commit Workflow

`git status` → `git add .` → `git commit -m "Commit Message"` → `git push`

### 2. Branch Management Workflow

`git switch -c Feature_Branch` → Develop & Commit → `git push -u origin Feature_Branch` → Switch to main branch `git switch main` → `git pull origin main` → `git merge --no-ff Feature_Branch` → `git push origin main` → `git branch -d Feature_Branch` + `git push origin --delete Feature_Branch`

### 3. Common Emergency Commands

- Undo working directory changes: `git restore filename`

- Rollback local unpushed commits: `git reset --soft Commit_ID`

- Rollback remote pushed commits: `git revert Commit_ID`

## VIII. Important Notes

1. DO NOT develop directly on the main branch (main/master), all features must be merged after being completed on sub-branches;

2. Before pushing code, be sure to pull the latest remote code to avoid conflicts;

3. For code already pushed to remote, **ABSOLUTELY DO NOT** use `git reset --hard` to force rollback;

4. Force push (`git push -f`) and force delete branch (`git branch -D`) are only for emergency situations, need to communicate with the team in advance;

5. Commit messages and branch naming should follow team conventions, for easy collaboration and traceability.

## Stash Code

```bash

git stash
```


## Cancel file tracking
git rm --cached fileName
