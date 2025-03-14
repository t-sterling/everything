# 🛠️ Git Comprehensive Cheat Sheet

## **1. Basics**
Git is a distributed version control system that helps track changes in source code during software development.
```sh
git init                  # Initialize a new Git repository
git clone <repo-url>      # Clone a repository
git status                # Show status of working directory and staging area
git add <file>            # Add a file to staging
git commit -m "message"   # Commit changes
git push                  # Push commits to remote repository
git pull                  # Fetch and merge latest changes
```

---

## **2. Advanced Logging & History Tracking**
Git provides powerful logging and history tracking tools to see who changed what and when.

### **Who did what and when?**
```sh
git log --author="John"                          # Show commits by a specific author
git log --since="1 week ago"                     # Show commits from the last week
git log --grep="fix"                             # Show commits with messages containing "fix"
git log --oneline --graph --decorate --all      # View history as a graph
git blame <file>                                 # Show last modification for each line of a file
git shortlog -sn                                 # Show commit count by author
```

### **Finding When Something Changed**
```sh
git log -p -S'specific code snippet' <file>      # Find commits that added/removed a specific string
git diff <commit1> <commit2>                     # Show changes between two commits
git show <commit>                                # Show details of a specific commit
```

---

## **3. Debugging & Finding Faulty Commits**
Git provides tools like bisect to find the exact commit that introduced an issue.

### **Using Git Bisect**
`git bisect` helps find the commit that introduced a bug by performing a binary search through commit history.
```sh
git bisect start               # Start bisecting
git bisect bad                 # Mark current commit as bad
git bisect good <commit>        # Mark a known good commit
git bisect run <script>        # Automate bisecting using a test script
git bisect reset               # Reset after finding the bad commit
```

### **Reverting Changes**
```sh
git revert <commit>             # Create a new commit that undoes changes
git reset --hard <commit>       # Reset to a previous commit and discard all changes
git reflog                      # Show history of branch movements
```

---

## **4. Working with Branches**
Branches allow multiple developers to work on different features simultaneously.
```sh
git branch                      # List branches
git branch <new-branch>         # Create a new branch
git checkout <branch>           # Switch to another branch
git switch <branch>             # Alternative to checkout (modern Git)
git merge <branch>              # Merge a branch into the current branch
git rebase <branch>             # Reapply commits on top of another branch
git cherry-pick <commit>        # Apply a specific commit to another branch
git branch -d <branch>          # Delete a branch
```

---

## **5. Working with Remotes**
Remotes allow collaboration by syncing repositories between local and remote servers.
```sh
git remote -v                   # Show remotes
git remote add origin <url>      # Add a remote repository
git fetch origin                 # Fetch changes from remote
git push origin <branch>         # Push branch to remote
git pull origin <branch>         # Pull latest changes from remote
git remote prune origin          # Remove stale remote branches
```

---

## **6. Stashing & Managing Changes**
Stashing allows temporary storage of uncommitted changes to work on something else.
```sh
git stash                        # Save changes without committing
git stash list                   # List stashed changes
git stash apply                   # Apply latest stash without deleting it
git stash pop                    # Apply and remove latest stash
git stash drop                   # Delete latest stash
git stash clear                   # Remove all stashes
```

---

## **7. Interactive Rebase & Rewriting History**
Rebasing allows modifying commit history to maintain a clean project history.
```sh
git rebase -i HEAD~5             # Interactive rebase last 5 commits
git commit --amend               # Modify last commit message
git reset --soft HEAD~1          # Undo last commit but keep changes staged
git filter-branch --tree-filter 'rm -rf secrets.txt' HEAD  # Remove sensitive files from history
```

---

## **8. Managing Tags**
Tags are used to mark specific points in commit history, such as releases.
```sh
git tag                           # List all tags
git tag -a v1.0 -m "Version 1.0"  # Create an annotated tag
git tag v1.0                      # Create a lightweight tag
git push origin v1.0              # Push a tag to remote
git push origin --tags            # Push all tags
git tag -d v1.0                   # Delete a tag locally
git push origin --delete v1.0     # Delete a remote tag
```

---

## **9. Submodules**
Git submodules allow including one repository inside another as a dependency.
```sh
git submodule add <repo-url> path/to/submodule  # Add a submodule
git submodule init                              # Initialize submodules
git submodule update                            # Update submodules
git submodule foreach git pull                  # Update all submodules
git submodule deinit path/to/submodule          # Remove a submodule
```

---

## **10. Git Hooks (Automation)**
Git hooks allow automation of tasks such as linting or testing before committing.
```sh
cd .git/hooks               # Navigate to hooks folder
echo "echo Hello" > pre-commit  # Create a simple pre-commit hook
chmod +x pre-commit         # Make it executable
```

---

## **11. Cleaning Up & Maintenance**
Git provides garbage collection and cleanup tools to optimize repositories.
```sh
git gc --prune=now         # Garbage collect unreferenced objects
git prune                  # Remove orphaned objects
git fsck                   # Verify repository integrity
git repack -a -d           # Compress objects to optimize storage
```

---

## **12. Saving & Recovering Work**
Recover lost changes or commits using `reflog` and `reset`.
```sh
git reflog                  # Show all movements in the repository
git checkout HEAD@{3}       # Restore repo state from 3 actions ago
git checkout -- <file>      # Discard changes in a file
git reset --hard HEAD       # Discard all uncommitted changes
```

---

### **Happy Coding! 🚀**
