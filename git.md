# Git Notes

## SSH Keys and GitHub Branches

Go to main `.ssh` folder and `ssh-keygen` if required and copy the pub key to setting and ssh key

GitHub branches are like sections where dev or others push their code and then merge with main branch

## GitHub Token

To generate a token go to setting and scroll down to developer setting and go to personal access token and classic token and create a classic token. This is used for login and credentials

## Git Fundamentals

In GitHub we say changes as commit

When ever we make a change first we add and then we commit

To configure git we use:

First in a folder we use `git init` to initialize git:

```bash
git config --global user.name "santoshpalla27"
git config --global user.email "santoshpalla2002@gmail.com"
git config --list  # to list the credentials
```

For initial setup:

```bash
cd my-project/
git init                       # if not a Git repo yet
git remote add origin https://gitlab.com/your-username/your-repo.git
git add .
git commit -m "Initial commit"
git branch -M main             # Rename default branch to main (optional)
git push -u origin main
```

This is only used for first time

## Git Stages

We have three stages in git:
1. Working area
2. Staging area
3. Commits

### 1) Working Area
This is where the files are modified and changes are made

### 2) Staging Area
This stage is where the changed files are added for commit so you can only select or pick the files you want to commit to the staging area

### 3) Commit
This stage the stages files are commited (saved) with a message so we can track the changes and revert back to the specific commit

## Git HEAD

Git HEAD is where you are in the current repository it usually points to last commit and moves with you when you switch branch or revert things

## Git Commands

```bash
git clone <link_of_git_repo>  # to clone repo to local

git clone -b <branch-name> <repo-url>  # to clone a branch in the repo

git add . or filename  # to add new files or changes into staging state

git commit -m "some message"  # to commit the changes

git status  # to know the status of the code

git log  # to view the log files which has info about commits and commit id and time etc

git log --oneline  # to get one line logs

git show <commit_id>  # to view which files are commited

git checkout <commit_id>  # to replace the files state to where the files are in with the commit id -| to go back -- git checkout main

git checkout <commit_id>  # Your entire working directory (all files tracked in Git at that commit) is updated to look exactly the way things were at that point in history.

git push origin main  # to push the changes to the main branch or give branch name you want to push the changes

git pull --rebase  # pulls the central files to local and merges them
```

You have to go to setting and developer setting and generate a token and while pushing the changes use username and token as credentials

## Git Restoration

```bash
git restore <filename>  # Undo all uncommitted changes to this file and reset it to how it was in the latest commit.

git restore --staged .  # to undo the changes that are in staging or after git add . after this can use above command to undo changes

git revert <commit_id>  # this will revert the changes given with id and makes as a new commit | it revert only revert the changes made in that commit and make a new commit and only the data modified in that commit will be changed

git checkout -- <filename>  # This command discards any changes you made to that file since the last commit.

git reset --soft HEAD~1  # Undo last commit but keep changes staged

git reset --hard HEAD~1  # Undo last commit and discard all changes

git reset --hard <log_id>  # to take back code to that stage undo all commits and take to that commit code

git reset  # To unstage all files you added after git add .
```

For shared branches like main or develop → use `git revert` instead — it's team-safe.

### After reset — do we need to push?

The answer is:
- Yes, BUT with caution — and often you need `--force` (or better: `--force-with-lease`).

Unlike git revert, which is safe and creates a new commit, git reset rewrites history — so pushing after it can be dangerous if others are using the branch.

## Git Stash

When you run git stash, Git:
- Records the changes in your working directory (modified tracked files) and staging area (index).
- Reverts your working directory back to match the latest commit (so it's clean).
- Saves those recorded changes in a special "stash stack" (like a hidden pile of temporary commits).

This is useful when:
- You're in the middle of work, but need to switch branches to fix a bug.
- You don't want to commit half-baked code yet.

### Stash Commands

```bash
git stash  # Saves tracked changes (both staged and unstaged). Leaves the workspace clean.

git stash --staged  # save only staged files

git stash -u  # Include untracked files

git stash --all  # Stashes tracked, untracked, and ignored files.

git stash push -m "WIP: add login form UI"  # Stash with description

git stash list  # list stash

git stash show  # See what's inside a stash

git stash apply  # Re-applies last stash (stash@{0}), but does not remove it from the stack. Good if you want to "test" it or reuse later.

git stash apply stash@{2}  # re-apply specific stash

git stash pop  # Re-applies last stash and removes it from stack.
```

### Common Workflow Example

```bash
# working on main, halfway through a feature
git stash -u             # save my messy changes including untracked
git checkout production  # switch branch to fix an urgent bug
# fix, commit, push
git checkout main
git stash pop            # bring back feature work
```

## Git Logging

```bash
git log -num  # to view last n number of logs

git log -p -2  # to view the changes made and no of logs

git log --stat  # to view the summary of the changes

git log --author <nameofauthor>  # to get logs of an author

git log -number  # to get those number of last logs

git log --since "date"  # to get logs since that particular date yyy-mm-dd

git log --author <name> --since "date"  # to get the logs of author since that date

git log --author <name> --since "date" --oneline  # to get the logs of author since that date in oneline
```

## Git Reflog

### What Git Reflog Does

- Shows the history of HEAD changes: Every time you check out a branch, make a new commit, rebase, reset, merge, or even accidentally wander through Git's wilderness, reflog records the event.
- Local only: This log is stored only in your local repo. Remote repos (like GitHub) don't see your reflog.
- Lifesaver after mistakes: If you reset --hard and panic because commits are "gone," reflog can help you find the commit hash and bring it back.

### Commands

```bash
git reflog  # Example Output

3b2c1d7 HEAD@{0}: commit: fix typo in readme
9f8e6a4 HEAD@{1}: rebase finished: returning to refs/heads/main
9f8e6a4 HEAD@{2}: checkout: moving from feature/login to main
d41e9ff HEAD@{3}: commit (amend): update login form

git reset --hard <commit_id>  # Undo an accidental reset

# Find lost commits (after rebases or branch deletions):
git reflog show feature/login

# Recover a deleted branch:
# Let's say you delete feature/login but realize it had important commits. You can find its last commit via:
git reflog
git checkout -b feature/login <commit_hash>
```

To make other work on my GitHub repo we need to add to them in our collaborator you can give access by going to the repo and go to collaborators and add people and they need to accept the invitation now they can also push the code

## Branch Management

Git branch is like a pointer where we create a new branch on a commit and then work of that branch and add changes and make commits then merge the working branch commit to the main branch where the code taken from or want to update by merge or pr

Git HEAD is where you are in the current repository it usually points to last commit and moves with you when you switch branch or revert things

### Branch Commands

```bash
git branch  # to check the status of the branch or the no of branches available

git branch <name>  # to create the branch

git checkout <branch_name>  # to switch branch

git checkout -b <branch_name>  # to create a new branch and switch to it

git switch feature-branch  # new alternative to switch the branch

git checkout <branch_name>  # to go to branch when there is file with same name as branch

git branch -d <name>  # to delete the branch

git diff <branch_name>  # to compare two branches

git merge <branch_name>  # takes the commits from the given branch and combines them into your current branch. after merge need to push changes again

git push origin <branch_name>  # to push code from to existing repo with new branch

git branch -r  # to view the branches after cloning repo
```

## Git Branching Strategy

### Why Do We Need Gitflow Branching Strategy?

When teams are small (1–2 devs), you can survive on "commit to main" 😅.
But when you grow to 5, 10, 50+ developers, multiple features, urgent fixes, releases happening in parallel — chaos starts.
If you don't have a strong branching strategy:

- 🚨 Hotfixes will collide with feature releases.
- 🐛 Bug fixes might accidentally go to production unfinished.
- 🚀 Feature delivery slows down because everyone fights over merge conflicts.
- 🤕 Release management becomes hell: what should go live, what should not?

- Merge commit → A special commit automatically created by Git when merging branches (e.g., after git pull) to combine both histories.
- Normal commit → The commit you create manually after adding or changing files (git add . && git commit -m "msg").
- Even on the same branch, if your local and remote have diverged and you resolve conflicts, Git creates a merge commit to record that merge. and it includes our changes as well

Gitflow brings discipline and clear separation between:

### Typical Workflow (Step-by-Step)

1. Update main:
   ```bash
   git checkout main
   git pull origin main
   ```

2. Create your branch:
   ```bash
   git checkout -b feature/add-profile-page
   ```

3. Do your work (code, commit, etc.):
   ```bash
   git add .
   git commit -m "Add profile page UI"
   ```

4. Push your branch to GitHub:
   ```bash
   git push origin feature/add-profile-page
   ```

5. Go to GitHub → Create Pull Request (PR)
   Ask teammates to review your code.

6. After approval → Merge PR → Delete branch (optional)

### Alternative: CLI Commands

```bash
git checkout main
git pull origin main
git merge feature/login         # Git will try to automatically combine the changes.
```

#### ⚠️ If there are MERGE CONFLICTS…

Git will tell you which files have conflicts. Open those files — you'll see something like:

```
<<<<<<< HEAD
This is from main branch
=======
This is from feature/login branch
>>>>>>> feature/login
```

Fix it manually:

- Decide what the final code should be.
- Delete the conflict markers (<<<<<<<, =======, >>>>>>>).
- Save the file.

Then mark it as resolved:
```bash
git add <filename>         # e.g. git add index.html
```

#### 💾 Complete the merge with a commit (if needed)

If Git didn't auto-commit (because of conflicts), run:

```bash
git commit -m "Merge feature/login into main"
```

If there were no conflicts, Git usually creates a merge commit automatically.

```bash
git push origin main
```

### Back on working branch, update branch again:

```bash
git checkout branch
git pull origin branch
```

### Why It's Risky (The Problem)

If 2+ devs push changes to the same branch:

- You might overwrite each other's work.
- You might get merge conflicts (Git doesn't know which code to keep).
- Code might break because someone pushed half-finished work.

✅ Best practice: Each person should work on their own feature branch → then merge into shared branch via Pull Request.

### How to Handle Multiple People Working on the Same Branch

#### Step-by-Step Workflow (Safe Collaboration)

1. 🔁 Always Pull Before You Start Working

   Before writing any code, pull the latest version of the branch:

   ```bash
   git checkout shared-branch-name
   git pull origin shared-branch-name
   ```

   This ensures you're not working on old code.

2. 🔄 Pull Frequently While Working

   Don't wait until you finish everything! Pull every few hours (or before big commits):

   ```bash
   git pull origin shared-branch-name
   ```

   Catches others' changes early → easier to fix conflicts.

3. 💾 Commit + Push Small, Often

   Instead of one giant "I did everything" commit, do small logical commits:

   ```bash
   git add .
   git commit -m "Add login validation"
   git push origin shared-branch-name
   ```

   Smaller = easier to merge, review, and rollback if needed.

4. ⚠️ If You Get a Merge Conflict

   Sometimes Git says: "I don't know which code to keep." That's a merge conflict.

   ➤ Example:
   You changed line 10 to say "Hello", but your teammate changed it to "Hi".

   ➤ How to fix:
   - Git will mark the file with <<<<<<<, =======, >>>>>>>.
   - Open the file, decide what the final code should be.
   - Remove the conflict markers.
   - Save → add → commit → push.

### Real-Life Example

Team is working on feature/user-profile together.

👨‍💻 Alice:
```bash
git checkout feature/user-profile
git pull origin feature/user-profile
# edits profile.js → commits → pushes
```

👩‍💻 Bob (at same time):
```bash
git checkout feature/user-profile
git pull origin feature/user-profile
# edits profile.js → tries to push → gets rejected!
# He must pull again → fix conflict → then push
```

Result: Both changes are safely merged.

## Git Fetch, Pulling and Rebase

Never use git rebase in shared branch and even in your own branch use carefully

### ✅ Rule of thumb

Rebase only on private branches or branches no one else is working on.
On shared branches, always merge to avoid rewriting history.

Git pull fetch or rebase can be done while working on same branch and also when you want to be updated with the main branch or the remote branch commits

Git pull and pull --rebase does same things but pull creates a merge commit while in --rebase we apply remote commits first to the local and apply our commit on top of it by solving any merge conflicts if any

### Git Fetch

```bash
git fetch  # is used to pull the changes in the remote repo but not change the local code

git diff origin/main  # to see the changes in the remote code and the local code

git merge origin/main  # to merge the changes of the remote repo with local code

git push origin main  # to push the changes

git merge --abort  # to abort the merge

git pull origin main  # to pull the latest changes and merge while fetch require manual merge

git pull --rebase  # revises the remote commits on working commits as if the changes you changes are made after those commits

git pull --rebase → rebase onto the branch your current branch is tracking.

git pull --rebase origin main → rebase your current branch onto origin/main (use this when you want to update a feature branch with the latest main).
```

### Total Process

#### For Git Merge

```bash
git merge feature-branch

# If conflict happens:
git status                  # see files
nano file.txt               # or use VSCode/editor to fix
# fix content, remove <<<<<< ==== >>>>>>
git add file.txt            # mark resolved

git status # to check
# Repeat for all conflict files
git commit                  # finish merge
git push origin master      # share it
```

#### Git Pull --rebase

```
Successfully rebased and updated refs/heads/master.
```

If any conflict solve them:

```bash
git status  # to check

git rebase --continue  # run this until everything is solved and should see successfully rebased

git add file and commit and push
```

### Git Pull Origin Main vs Git Pull --rebase

**Read carefully important**

While git pull or fetch applies newly commited changes on top of the working commit so all changes will be added at once while | git pull --rebase pulls the remote changes and if there are any commits before you by other then it will apply those commits one by one on the local repo so it looks as if you applied changes after the others commits so the working tree will be clean and there is still chance of having a merge confict but the impact will be less as you will be solving conflicts one commit after another instead like git pull where all the commits are added at once

### Option 2: Rebase Testing onto Staging (Advanced / Cleaner History) (hard)

Rebasing rewrites your testing branch to appear as if you started from the latest staging.

⚠️ Only use this if YOU are the only one working on testing, or you coordinate with your team — because it rewrites history!

```bash
# 1. Make sure testing is up to date locally
git checkout testing
git pull origin testing

# 2. Rebase testing onto latest staging
git rebase staging

# 3. If conflicts → fix them → then continue
git add .
git rebase --continue
# Repeat until done

# 4. Force push (because history changed)
git push origin testing --force-with-lease
```

What happens?
- Your testing branch now sits "on top of" the latest staging.
- History looks cleaner — no extra "merge commit".
- BUT — if others pulled the old testing, they'll get confused unless they reset.

✅ Use this if:
- You're solo on the branch
- You want clean, linear history
- You understand force pushing risks

## Merge Conflicts

```bash
git merge --abort   # aborts the merge or pull
```

When you are merging two repo which has same file name but different or different timeline it becomes hard to merge in that case:-
When the code is merged the both code will merge and the code has two branch code and the dev will decide which code to keep

### Handling Merge Conflicts

If Git cannot automatically merge changes, a merge conflict occurs.

### Resolving Merge Conflicts

1. Identify conflicts using: `git status`

2. Open conflicting files and manually edit sections marked by `<<<<<<<` and `>>>>>>>`.

#### What you actually do

- Open the file → Git has paused here for you to fix it.
- Choose what should stay:
  - Sometimes you take only your code.
  - Sometimes only theirs.
  - Sometimes you combine bits of both! (This is the most common in team projects.)
- Remove the markers (<<<<<<<, =======, >>>>>>>).
  - They are not real code — just Git's way of saying "here's the fight—resolve it."
  - Leave the file as clean, working code.
- Save the file.
- Stage the resolved file: `git add filename`
- `git status`
- Continue your merge or rebase:
  - `git merge --continue`
  - OR
  - `git rebase --continue`

## Pull Requests and Code Reviews

A pull request (PR) is a request to merge changes into a remote branch.

You can see the changes made and add additional message and also add reviewers, labels etc

### Creating a Pull Request on GitHub

1. Push your branch to GitHub: `git push origin feature_branch`
2. Go to GitHub and open a pull request.
3. Request a review and discuss changes.
4. Once approved, merge the pull request.

Fork -- rough copy

```bash
git tag  # to list the tags

git tag <tag_name>  # to tag the last commit with tag_name

git tag <tag_name> <log_id>  # to tag a specific log

git checkout <tag_name>  # to go into the log of specific name and can access files using ls

git push origin <tag_name>  # to push the tags and can be view in tags in GitHub and that has only limited code toll the tag

git push origin -d <tag_name>  # to delete the tag in central repo

git checkout -b <branch-name> <tag-name>  # to create a branch with tag id
```

Git branch protection rules
- Go to setting and branches

## GitHub Hook

Go to setting and web hook and add http://<jenkins-domain>/github-webhook/

And content type application Json

And select all events or push by choice and create

## What is Git Cherry-Pick?

Git cherry-pick is a Git command that lets you take a specific commit (or commits) from one branch and apply it onto another branch.

### When to Use Cherry-Pick:

- When you want to apply specific bug fixes from a development branch to the main branch without merging all changes.
- To selectively move commits between branches.

### How to Use It:

1. Find the commit hash (ID) you want to cherry-pick, e.g., a1b2c3d.
2. Switch to the branch where you want to apply the commit:

   ```bash
   git checkout <working_where_u_want_to_push_the_changes>
   git cherry-pick a1b2c3d
   ```

3. (Optional) Resolve conflicts if they occur

   Git will stop and mark conflicted files.

   Open conflicted files, look for markers like:

4. Stage the resolved files

   ```bash
   git add <file1> <file2> ...
   ```

5. Continue the cherry-pick process

   ```bash
   git cherry-pick --continue
   ```

6. Push the changes to the remote repository

   ```bash
   git push origin target-branch
   ```

Example workflow:

```bash
git checkout target-branch
git cherry-pick <commit-hash>

# If conflicts happen:
# resolve conflicts in files
git add <resolved-files>
git cherry-pick --continue

# If you want to abort cherry-pick:
# git cherry-pick --abort

commit and git push origin target-branch
```

Like this will only take the committed files from the branch at that commit

### Summary:
- Normally, git cherry-pick applies the commit and creates a new commit automatically.
- If conflicts happen, resolve them, stage files, and run git cherry-pick --continue to finish.

## Git Ignore

`.gitignore` -- will be the file name

The files named in the gitignore will not be pushed to the central repo

Format of files every file should be in new line

```
index.html
styles.css
*.html
etc
```