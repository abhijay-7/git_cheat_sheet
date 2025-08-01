# Git/GitHub Command Cheat Sheet

## Repository Setup & Configuration

### Initialize Repository
- `git init` - Initialize a new Git repository
- `git clone <url>` - Clone a remote repository
- `git clone <url> <directory>` - Clone into specific directory

### Configuration
- `git config --global user.name "Your Name"` - Set global username
- `git config --global user.email "your.email@example.com"` - Set global email
- `git config --list` - View all configuration settings
- `git config user.name` - View current username

## Adding & Staging

### Add Files
- `git add <file>` - Stage a specific file
- `git add .` - Stage all files in current directory
- `git add -A` - Stage all files (including deleted)
- `git add *.js` - Stage all JavaScript files
- `git add -p` - Interactive staging (choose hunks)

### Unstage Files
- `git reset <file>` - Unstage a specific file
- `git reset` - Unstage all files
- `git restore --staged <file>` - Unstage a file (Git 2.23+)

## Committing

### Create Commits
- `git commit -m "message"` - Commit with message
- `git commit -am "message"` - Stage and commit all tracked files
- `git commit --amend` - Modify the last commit
- `git commit --amend -m "new message"` - Change last commit message
- `git commit --no-edit --amend` - Add changes to last commit without changing message

## Branching

### Create Branches
- `git branch <branch-name>` - Create new branch
- `git checkout -b <branch-name>` - Create and switch to new branch
- `git switch -c <branch-name>` - Create and switch to new branch (Git 2.23+)

### Switch Branches
- `git checkout <branch-name>` - Switch to existing branch
- `git switch <branch-name>` - Switch to existing branch (Git 2.23+)
- `git checkout -` - Switch to previous branch

### List Branches
- `git branch` - List local branches
- `git branch -r` - List remote branches
- `git branch -a` - List all branches (local and remote)
- `git branch -v` - List branches with last commit info

## Deleting

### Delete Files
- `git rm <file>` - Remove file from working directory and stage deletion
- `git rm --cached <file>` - Remove file from Git but keep in working directory
- `git rm -r <directory>` - Remove directory recursively

### Delete Branches
- `git branch -d <branch-name>` - Delete merged branch (safe)
- `git branch -D <branch-name>` - Force delete branch (even if unmerged)
- `git push origin --delete <branch-name>` - Delete remote branch

### Delete Commits
- `git reset --hard HEAD~1` - Delete last commit (dangerous - loses changes)
- `git reset --soft HEAD~1` - Undo last commit but keep changes staged
- `git reset HEAD~1` - Undo last commit and unstage changes
- `git revert <commit-hash>` - Create new commit that undoes specified commit

### Remove Files from Commit History (Sensitive Data)
- `git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch <file>' --prune-empty --tag-name-filter cat -- --all` - Remove file from all commits (old method)
- `git filter-repo --path <file> --invert-paths` - Remove file from history (requires git-filter-repo)
- `git filter-repo --strip-blobs-bigger-than 10M` - Remove large files from history

### BFG Repo-Cleaner (Recommended for large cleanups)
```bash
# Install BFG first: https://rtyley.github.io/bfg-repo-cleaner/
java -jar bfg.jar --delete-files <filename> <repo.git>
java -jar bfg.jar --strip-blobs-bigger-than 50M <repo.git>
java -jar bfg.jar --delete-folders .git <repo.git>
```

### After History Cleanup (Required Steps)
- `git reflog expire --expire=now --all` - Expire reflog entries
- `git gc --prune=now --aggressive` - Garbage collect and prune
- `git push --force-with-lease` - Force push cleaned history (DANGEROUS)

### Remove Sensitive Files Step-by-Step
1. **Using git-filter-repo (recommended)**:
   ```bash
   # Install: pip install git-filter-repo
   git filter-repo --path .env --invert-paths
   git filter-repo --path config/secrets.yml --invert-paths
   ```

2. **Using filter-branch (legacy)**:
   ```bash
   git filter-branch --force --index-filter \
     'git rm --cached --ignore-unmatch .env' \
     --prune-empty --tag-name-filter cat -- --all
   ```

3. **Clean up and force push**:
   ```bash
   git reflog expire --expire=now --all
   git gc --prune=now --aggressive
   git push --force-with-lease --all
   git push --force-with-lease --tags
   ```

### Important Notes for History Rewriting
- ⚠️ **DANGER**: These operations rewrite history and change commit hashes
- ⚠️ **Coordinate with team**: All collaborators need to re-clone after force push
- ⚠️ **Backup first**: Create backup branch before running these commands
- ⚠️ **GitHub/GitLab**: May need to contact support to purge from their caches
- ✅ **Add to .gitignore**: Prevent future accidents

## Viewing & Inspecting

### View Status & History
- `git status` - Show working directory status
- `git log` - View commit history
- `git log --oneline` - Compact commit history
- `git log --graph` - Visual commit history
- `git log -p` - Show commit history with diffs
- `git log --author="name"` - Filter commits by author

### View Changes
- `git diff` - Show unstaged changes
- `git diff --staged` - Show staged changes
- `git diff <commit1> <commit2>` - Compare two commits
- `git show <commit-hash>` - Show specific commit details
- `git blame <file>` - Show who changed each line in a file

## Searching & Finding

### Search in Files
- `git grep "search-term"` - Search for string in tracked files
- `git grep "search-term" <commit-hash>` - Search for string in specific commit
- `git grep -n "search-term"` - Search with line numbers
- `git grep -i "search-term"` - Case-insensitive search
- `git grep -l "search-term"` - Show only filenames with matches
- `git grep "search-term" -- "*.js"` - Search only in JavaScript files
- `git grep -E "regex-pattern"` - Search using regular expressions

### Find Commits Containing String
- `git log -S "search-term"` - Find commits that added/removed the string
- `git log -G "regex-pattern"` - Find commits where regex matches added/removed lines
- `git log --grep="commit-message-text"` - Find commits by commit message
- `git log -p -S "search-term"` - Show commits and diffs containing string
- `git log --all --grep="bug fix"` - Search commit messages across all branches

### Find Files and Filenames
- `git ls-files | grep "filename"` - Find files by name in current branch
- `git log --name-only --oneline | grep "filename"` - Find commits that touched specific file
- `git log --follow -- <filename>` - Show history of a file (even through renames)
- `git log --stat --oneline | grep "filename"` - Find files with change statistics
- `git show --name-only <commit-hash>` - Show files changed in specific commit
- `git diff-tree --no-commit-id --name-only -r <commit-hash>` - List files in commit

### Advanced Search
- `git log --pickaxe-regex -S "regex-pattern"` - Use regex with -S option
- `git log --source --all --grep="pattern"` - Search across all branches with source info
- `git log --since="2023-01-01" --until="2023-12-31" -S "search-term"` - Search in date range
- `git log --author="John" -S "search-term"` - Search by author and content

## Time Travel & State Management

### Go to Specific Commit State
- `git checkout <commit-hash>` - Move to specific commit (detached HEAD)
- `git checkout <commit-hash> .` - Update working directory to commit state
- `git reset --hard <commit-hash>` - Reset current branch to specific commit (destructive)
- `git checkout <commit-hash> -- <file>` - Restore specific file from commit
- `git worktree add <path> <commit-hash>` - Create separate working directory at commit

### Navigate History
- `git checkout HEAD~1` - Go back one commit
- `git checkout HEAD~3` - Go back three commits
- `git checkout main@{yesterday}` - Go to main branch as it was yesterday
- `git checkout main@{2.hours.ago}` - Go to main branch 2 hours ago
- `git reflog` - Show history of HEAD movements (great for recovery)

### Temporary State Changes
- `git stash` - Save current changes temporarily
- `git checkout <commit-hash>` - Explore old state
- `git checkout -` - Return to previous branch/commit
- `git stash pop` - Restore saved changes

## Remote Repositories

### Add Remotes
- `git remote add origin <url>` - Add remote repository
- `git remote -v` - List remote repositories
- `git remote rename <old> <new>` - Rename remote
- `git remote remove <name>` - Remove remote

### Push & Pull
- `git push origin <branch>` - Push branch to remote
- `git push -u origin <branch>` - Push and set upstream branch
- `git push --all` - Push all branches
- `git pull` - Fetch and merge from remote
- `git pull origin <branch>` - Pull specific branch
- `git fetch` - Download changes without merging
- `git fetch --all` - Fetch all remotes

## Merging & Rebasing

### Merge
- `git merge <branch>` - Merge branch into current branch
- `git merge --no-ff <branch>` - Create merge commit even for fast-forward
- `git merge --squash <branch>` - Squash commits before merging

### Rebase
- `git rebase <branch>` - Rebase current branch onto another
- `git rebase -i HEAD~3` - Interactive rebase of last 3 commits
- `git rebase --continue` - Continue rebase after resolving conflicts
- `git rebase --abort` - Abort rebase operation

## Conflict Resolution

### During Merge/Rebase Conflicts
- `git status` - Show conflicted files
- `git diff` - Show conflict markers in files
- `git add <file>` - Mark conflict as resolved after manual edit
- `git merge --continue` - Continue merge after resolving conflicts
- `git merge --abort` - Abort merge and return to pre-merge state
- `git rebase --continue` - Continue rebase after resolving conflicts
- `git rebase --skip` - Skip current commit during rebase
- `git rebase --abort` - Abort rebase and return to original branch

### Conflict Resolution Tools
- `git mergetool` - Launch configured merge tool
- `git mergetool --tool=vimdiff` - Use specific merge tool
- `git config --global merge.tool <tool>` - Set default merge tool
- `git checkout --ours <file>` - Keep your version during conflict
- `git checkout --theirs <file>` - Keep their version during conflict

### Manual Conflict Resolution
```bash
# Edit files to resolve conflicts (remove <<<<<<< ======= >>>>>>> markers)
# Then:
git add <resolved-file>
git commit  # (for merge) or git rebase --continue (for rebase)
```

## Master/Main Branch Operations

### Branch Naming & Default Branch
- `git config --global init.defaultBranch main` - Set default branch name for new repos
- `git branch -m master main` - Rename master to main locally
- `git push -u origin main` - Push new main branch to remote
- `git push origin --delete master` - Delete old master branch from remote
- `git symbolic-ref refs/remotes/origin/HEAD refs/remotes/origin/main` - Update remote HEAD

### Change Default Branch on Remote
```bash
# On GitHub/GitLab: Change default branch in repository settings first, then:
git fetch origin
git branch -u origin/main main
git remote set-head origin -a
```

### Protect Master/Main Branch
- Use GitHub/GitLab web interface for branch protection rules
- Enable "Require pull request reviews before merging"
- Enable "Require status checks to pass before merging"
- Enable "Restrict pushes to matching branches"

### Master Branch Workflows
- `git checkout main && git pull` - Update main branch
- `git checkout -b feature/new-feature` - Create feature branch from main
- `git checkout main && git merge feature-branch` - Merge to main
- `git push origin main` - Push updated main
- `git branch -d feature-branch` - Delete merged feature branch

## Stashing

### Save Work Temporarily
- `git stash` - Stash current changes
- `git stash save "message"` - Stash with custom message
- `git stash -u` - Stash including untracked files
- `git stash list` - List all stashes
- `git stash show` - Show stash contents
- `git stash pop` - Apply and remove latest stash
- `git stash apply` - Apply stash without removing it
- `git stash drop` - Delete latest stash
- `git stash clear` - Delete all stashes

## Undoing Changes

### Restore Files
- `git checkout -- <file>` - Discard changes in working directory
- `git restore <file>` - Discard changes in working directory (Git 2.23+)
- `git checkout <commit> -- <file>` - Restore file from specific commit

### Reset Operations
- `git reset --soft <commit>` - Reset to commit, keep changes staged
- `git reset --mixed <commit>` - Reset to commit, unstage changes (default)
- `git reset --hard <commit>` - Reset to commit, discard all changes

## Tagging

### Create Tags
- `git tag <tag-name>` - Create lightweight tag
- `git tag -a <tag-name> -m "message"` - Create annotated tag
- `git tag -a <tag-name> <commit-hash>` - Tag specific commit

### List & Delete Tags
- `git tag` - List all tags
- `git tag -d <tag-name>` - Delete local tag
- `git push origin --delete <tag-name>` - Delete remote tag
- `git push origin <tag-name>` - Push specific tag
- `git push --tags` - Push all tags

## GitHub Specific

### GitHub CLI (gh) - Repository Management
- `gh repo create` - Create new repository
- `gh repo clone <repo>` - Clone repository
- `gh repo view` - View repository details
- `gh repo edit --description "New description"` - Edit repository
- `gh repo delete <repo>` - Delete repository
- `gh repo fork <repo>` - Fork repository

### GitHub CLI - Pull Requests & Issues
- `gh pr create` - Create pull request
- `gh pr list` - List pull requests
- `gh pr view <number>` - View specific PR
- `gh pr merge <number>` - Merge pull request
- `gh pr close <number>` - Close pull request
- `gh issue create` - Create new issue
- `gh issue list` - List issues
- `gh issue view <number>` - View specific issue
- `gh issue close <number>` - Close issue

### Collaborator Management (GitHub CLI)
- `gh api repos/:owner/:repo/collaborators` - List collaborators
- `gh api --method PUT repos/:owner/:repo/collaborators/:username` - Add collaborator
- `gh api --method DELETE repos/:owner/:repo/collaborators/:username` - Remove collaborator
- `gh api repos/:owner/:repo/collaborators/:username/permission` - Check user permissions

### Advanced Collaborator Management
```bash
# Add collaborator with specific permission
gh api --method PUT repos/:owner/:repo/collaborators/:username \
  --field permission=push  # Options: pull, push, admin, maintain, triage

# Add collaborator to specific team (Organization repos)
gh api --method PUT orgs/:org/teams/:team/memberships/:username

# List repository permissions
gh api repos/:owner/:repo/collaborators --jq '.[].permissions'
```

### Branch Protection via CLI
```bash
# Enable branch protection
gh api --method PUT repos/:owner/:repo/branches/:branch/protection \
  --field required_status_checks='{"strict":true,"contexts":[]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1}' \
  --field restrictions=null

# List protected branches
gh api repos/:owner/:repo/branches --jq '.[] | select(.protected) | .name'

# Remove branch protection
gh api --method DELETE repos/:owner/:repo/branches/:branch/protection
```

### Team Management (Organizations)
```bash
# List organization teams
gh api orgs/:org/teams

# Create team
gh api --method POST orgs/:org/teams \
  --field name="Team Name" \
  --field description="Team description" \
  --field privacy=closed  # Options: secret, closed

# Add repository to team with permissions
gh api --method PUT orgs/:org/teams/:team/repos/:owner/:repo \
  --field permission=push  # Options: pull, push, admin, maintain, triage

# List team members
gh api orgs/:org/teams/:team/members
```

### Repository Access Control
```bash
# Set repository visibility
gh repo edit --visibility private  # Options: public, private, internal

# Transfer repository ownership
gh api --method POST repos/:owner/:repo/transfer \
  --field new_owner=":new-owner"

# Archive repository
gh repo archive <repo>

# Enable/disable features
gh repo edit --enable-issues=true
gh repo edit --enable-wiki=false
gh repo edit --enable-projects=true
```

### Webhook Management
```bash
# List webhooks
gh api repos/:owner/:repo/hooks

# Create webhook
gh api --method POST repos/:owner/:repo/hooks \
  --field name=web \
  --field config='{"url":"https://example.com/webhook","content_type":"json"}' \
  --field events='["push","pull_request"]'

# Delete webhook
gh api --method DELETE repos/:owner/:repo/hooks/:hook_id
```

### Working with Forks
- `git remote add upstream <original-repo-url>` - Add upstream remote
- `git fetch upstream` - Fetch from upstream
- `git merge upstream/main` - Merge upstream changes
- `gh repo sync` - Sync fork with upstream (GitHub CLI)

## Useful Aliases

Add these to your Git config for shortcuts:
```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual '!gitk'
```

## Emergency Commands

### When Things Go Wrong
- `git reflog` - Show reference log (find lost commits)
- `git fsck --lost-found` - Find dangling objects
- `git cherry-pick <commit-hash>` - Apply specific commit to current branch
- `git bisect start` - Start binary search for bug
- `git clean -fd` - Remove untracked files and directories (be careful!)

## .gitignore Usage

### Creating .gitignore
- Create `.gitignore` file in repository root
- Add patterns for files/folders to ignore
- Each line represents a pattern to ignore

### Common .gitignore Patterns
```gitignore
# Files
*.log
*.tmp
*.swp
secret.txt

# Directories
node_modules/
build/
dist/
.env/
temp/

# Operating System files
.DS_Store
Thumbs.db
desktop.ini

# IDE files
.vscode/
.idea/
*.sublime-*

# Languages/Frameworks
# Node.js
node_modules/
npm-debug.log
.env

# Python
__pycache__/
*.pyc
*.pyo
.env
venv/

# Java
*.class
target/
*.jar

# C/C++
*.o
*.exe
*.out
```

### .gitignore Commands
- `git check-ignore <file>` - Check if file is ignored and why
- `git check-ignore -v <file>` - Verbose output showing which rule applies
- `git add -f <ignored-file>` - Force add ignored file
- `git rm --cached <file>` - Remove already tracked file (to start ignoring it)
- `git clean -fdX` - Remove only ignored files
- `git status --ignored` - Show ignored files in status

### Global .gitignore
- `git config --global core.excludesfile ~/.gitignore_global` - Set global ignore file
- Create `~/.gitignore_global` with patterns to ignore in all repositories

### .gitignore Rules
- `file.txt` - Ignore specific file
- `*.log` - Ignore all .log files
- `temp/` - Ignore temp directory and all contents
- `/build` - Ignore build directory only in root
- `**/logs` - Ignore logs directory anywhere in project
- `doc/*.txt` - Ignore .txt files in doc directory (not subdirectories)
- `doc/**/*.txt` - Ignore .txt files in doc and all subdirectories
- `!important.log` - Don't ignore important.log (negation)
- `*.log` followed by `!debug.log` - Ignore all .log files except debug.log

## Best Practices

- Always pull before pushing
- Use meaningful commit messages
- Create feature branches for new work
- Review changes before committing
- Use `.gitignore` for files that shouldn't be tracked
- Never commit sensitive information (passwords, API keys)
- Commit frequently with small, logical changes
- Use `git status` often to understand current state
- Test your code before committing
- Write descriptive commit messages in present tense