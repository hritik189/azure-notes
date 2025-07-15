# Git & GitHub

## Introduction to Git

>**Git** is a distributed version control system that tracks changes in files and coordinates work among multiple developers. It allows you to:
- Track changes in your code over time
- Collaborate with others without conflicts
- Revert to previous versions when needed
- Work on different features simultaneously through branching

### Key Benefits of Git:
- **Distributed**: Every developer has a complete copy of the project history
- **Fast**: Most operations are local and lightning-fast
- **Secure**: Uses SHA-1 hashing to ensure data integrity
- **Flexible**: Supports various workflows for different team sizes

## Git Terminologies

### Core Concepts:

- **Repository (Repo)**: A project folder that contains all your code, files, and version history
- **Working Directory**: The current state of your project files on your local machine
- **Staging Area (Index)**: A temporary area where you prepare changes before committing
- **Commit**: A snapshot of your changes with a unique identifier (hash)
- **Branch**: A separate line of development, useful for working on features without affecting main code
- **HEAD**: A pointer to the current branch and commit you're working on
- **Origin**: The default name for the remote repository
- **Upstream**: The original repository you forked from
- **Fork**: A personal copy of someone else's repository
- **Merge**: Combining changes from one branch into another
- **Pull Request (PR)**: A request to merge code from one branch to another, often reviewed by teammates
- **Clone**: Creating a local copy of a remote repository
- **Remote**: A version of your repository hosted on a server (like GitHub)

### Git States:
- **Modified**: Files that have been changed but not yet staged
- **Staged**: Files that are ready to be committed
- **Committed**: Files that are safely stored in the Git database

## Introduction to GitHub

>**GitHub** is a cloud-based platform that hosts Git repositories and provides additional collaboration features:

### What GitHub Offers:
- **Repository Hosting**: Store your Git repositories in the cloud
- **Collaboration Tools**: Issues, Pull Requests, Discussions, and more
- **Project Management**: Project boards, milestones, and labels
- **CI/CD Integration**: GitHub Actions for automated workflows
- **Documentation**: README files, wikis, and GitHub Pages
- **Security Features**: Vulnerability scanning and dependency management
- **Social Coding**: Follow developers, star repositories, and contribute to open source

### GitHub vs Git:
- **Git**: Version control system (software)
- **GitHub**: Hosting service for Git repositories (platform)

## Git Configuration Commands

Set up your identity and preferences before working with Git:

```bash
# Set global username
git config --global user.name "Your Name"

# Set global email
git config --global user.email "your.email@example.com"

# Check current configuration
git config --list

# Check specific configuration
git config user.name
git config user.email

# Edit configuration file directly
git config --global --edit

# Set default branch name
git config --global init.defaultBranch main

# Set default editor
git config --global core.editor "code --wait"
```

## Repository Setup - Clone & Initialize

```bash
# Initialize a new repo in current directory
git init

# Initialize with specific branch name
git init -b main

# Clone a remote repository
git clone https://github.com/username/repo.git

# Clone into a specific directory
git clone https://github.com/username/repo.git my-project

# Clone a specific branch
git clone -b dev https://github.com/username/repo.git

# Clone with depth (shallow clone)
git clone --depth 1 https://github.com/username/repo.git

# Add remote to existing repository
git remote add origin https://github.com/username/repo.git

# View remote repositories
git remote -v

# Change remote URL
git remote set-url origin https://github.com/username/new-repo.git
```

## Add - Staging Changes

>The `git add` command moves changes from your working directory to the staging area:

```bash
# Check status of working directory
git status

# Add a specific file to staging area
git add filename.txt

# Add multiple files
git add file1.txt file2.txt

# Add all files in current directory
git add .

# Add all files in the repository
git add -A

# Add all modified files (not new files)
git add -u

# Add files interactively
git add -i

# Add parts of a file (patch mode)
git add -p filename.txt

# Remove file from staging area
git reset filename.txt

# Remove all files from staging area
git reset
```

## Commit - Saving Changes

`Commits create snapshots of your staged changes:`

```bash
# Commit staged changes with message
git commit -m "Add new feature"

# Commit with detailed message
git commit -m "Add user authentication" -m "Implemented login and signup functionality"

# Add and commit all tracked files in one step
git commit -am "Fix bug in user validation"

# Commit with editor for longer message
git commit

# Amend last commit (change message or add forgotten files)
git commit --amend -m "Corrected commit message"

# Commit only specific files
git commit filename.txt -m "Update specific file"

# View commit history
git log

# View commit history with graph
git log --graph --oneline

# View specific commit details
git show commit-hash
```

## Push/Pull - Syncing with Remote

### Pull - Getting Changes from Remote:
```bash
# Fetch and merge changes from remote branch
git pull origin main

# Pull with rebase instead of merge
git pull --rebase origin main

# Pull from specific branch
git pull origin develop

# Fetch changes without merging
git fetch origin

# Fetch all branches
git fetch --all
```

### Push - Sending Changes to Remote:
```bash
# Push local commits to remote branch
git push origin main

# Push new branch to remote and set upstream
git push -u origin feature-branch

# Push all branches
git push --all origin

# Push tags
git push --tags origin

# Force push (use with caution)
git push --force origin main

# Force push safely
git push --force-with-lease origin main

# Delete remote branch
git push origin --delete branch-name
```

## Branching - Managing Feature Development

`Branches allow you to work on different features simultaneously:`

```bash
# List all local branches
git branch

# List all branches (including remote)
git branch -a

# List remote branches only
git branch -r

# Create a new branch
git branch feature-branch

# Switch to a branch
git checkout feature-branch

# Create and switch to a new branch
git checkout -b feature-branch

# Switch to a branch (modern syntax)
git switch feature-branch

# Create and switch to new branch (modern syntax)
git switch -c feature-branch

# Rename current branch
git branch -m new-branch-name

# Rename any branch
git branch -m old-name new-name

# Delete a local branch
git branch -d feature-branch

# Force delete a local branch
git branch -D feature-branch

# Delete remote branch
git push origin --delete feature-branch

# Set upstream for current branch
git branch --set-upstream-to=origin/main

# Show branch tracking information
git branch -vv
```

### Merging Branches:
```bash
# Merge another branch into current branch
git merge feature-branch

# Merge with no fast-forward (creates merge commit)
git merge --no-ff feature-branch

# Merge with squash (combines all commits)
git merge --squash feature-branch

# Abort merge in case of conflicts
git merge --abort
```

## GitHub Issues

`GitHub Issues help track bugs, feature requests, and tasks:`

### Creating Issues:
- Navigate to the "Issues" tab in your repository
- Click "New Issue" button
- Choose from issue templates or create blank issue
- Add title, description, labels, assignees, and milestones

### Issue Features:
- **Labels**: Categorize issues (bug, enhancement, documentation, etc.)
- **Assignees**: Assign team members to work on issues
- **Milestones**: Group issues into release cycles
- **Projects**: Organize issues in project boards
- **References**: Link issues to commits, PRs, and other issues

### Working with Issues via Git:
```bash
# Reference issue in commit message
git commit -m "Fix login bug (fixes #123)"

# Close issue with commit
git commit -m "Add user profile feature (closes #456)"

# Reference multiple issues
git commit -m "Update documentation (refs #789, #101)"
```

### Issue Keywords:
- `fixes #123` - Closes issue when PR is merged
- `closes #123` - Closes issue when PR is merged
- `resolves #123` - Closes issue when PR is merged
- `refs #123` - References issue without closing

## GitHub Discussions

`GitHub Discussions provide a space for community conversations:`

### Types of Discussions:
- **Q&A**: Questions and answers
- **General**: Open-ended conversations
- **Ideas**: Feature requests and suggestions
- **Show and Tell**: Share what you've built
- **Polls**: Gather community input

### Discussion Features:
- **Categories**: Organize discussions by topic
- **Reactions**: React to posts with emojis
- **Answers**: Mark helpful responses in Q&A
- **Notifications**: Stay updated on discussions
- **Moderation**: Manage community guidelines

### Best Practices:
- Use appropriate categories for different types of discussions
- Search existing discussions before creating new ones
- Provide clear titles and descriptions
- Engage respectfully with the community
- Mark helpful answers in Q&A discussions

## Advanced Git Commands

### Revert/Reset - Undoing Mistakes:
```bash
# Undo last commit but keep changes staged
git reset --soft HEAD~1

# Undo last commit and unstage changes
git reset --mixed HEAD~1

# Discard changes and reset to last commit
git reset --hard HEAD~1

# Reset to specific commit
git reset --hard commit-hash

# Revert a specific commit (creates a new commit)
git revert HEAD

# Revert a specific commit by hash
git revert abc1234

# Revert merge commit
git revert -m 1 commit-hash
```

### Stashing Changes:
```bash
# Stash current changes
git stash

# Stash with message
git stash save "Work in progress on feature"

# List all stashes
git stash list

# Apply most recent stash
git stash apply

# Apply specific stash
git stash apply stash@{1}

# Apply and remove stash
git stash pop

# Drop specific stash
git stash drop stash@{1}

# Show stash contents
git stash show -p stash@{0}
```

### Tags:
```bash
# Create lightweight tag
git tag v1.0.0

# Create annotated tag
git tag -a v1.0.0 -m "Version 1.0.0 release"

# List all tags
git tag

# Show tag information
git show v1.0.0

# Push tags to remote
git push origin v1.0.0

# Push all tags
git push --tags

# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0
```

## GitHub Workflow Best Practices

### Pull Request Workflow:
1. Create a feature branch from main
2. Make changes and commit them
3. Push branch to GitHub
4. Create Pull Request
5. Request reviews from team members
6. Address feedback and make changes
7. Merge PR after approval
8. Delete feature branch

### Commit Message Best Practices:
- Use imperative mood: "Add feature" not "Added feature"
- Keep first line under 50 characters
- Separate subject from body with blank line
- Use body to explain what and why, not how
- Reference issues and PRs when relevant

### Branch Naming Conventions:
- `feature/user-authentication`
- `bugfix/login-error`
- `hotfix/critical-security-patch`
- `release/v1.2.0`

## Troubleshooting Common Issues

### Merge Conflicts:
```bash
# Check which files have conflicts
git status

# Edit conflicted files manually
# Look for conflict markers: <<<<<<<, =======, >>>>>>>

# After resolving conflicts
git add .
git commit -m "Resolve merge conflicts"

# Use merge tool
git mergetool
```

### Undoing Common Mistakes:
```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo changes to specific file
git checkout -- filename.txt

# Restore deleted file
git checkout HEAD -- filename.txt

# Unstage file
git reset HEAD filename.txt

# Change last commit message
git commit --amend -m "New message"
```

>This comprehensive guide covers all the essential Git and GitHub concepts you need to know for effective version control and collaboration!
