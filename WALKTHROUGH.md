# Stashing
Sometimes it's helpful to preserve your repo's working copy so that you can return to a clean state.
 * Perhaps you’ve pulled and gotten:
    `error: Your local changes to the following files would be overwritten by merge:`
 * Or perhaps you need to change branches to work on another feature and you don't want to commit or carry those changes into the other branch

```bash
git status -vv
git stash
# Saved working directory and index state WIP on master: 208baec add a .gitconfig as part of the versioned repo
git stash list
# stash@{0}: WIP on master: 208baec add a .gitconfig as part of the versioned repo
git status -vv              # We still have the untracked file in the working copy
git stash apply             # or pop
```

The standard case for stash is to not clear and save untracked files. 

```bash
git stash -u
git stash list
git stash pop
```

Stashes can have a descriptions

```bash
git stash save "Work on feature 123"
git stash list
git stash pop
```

Stashes can also be applied to on another branch. 

```bash
git stash branch new_feature  # if branch is existing you must check it out and pop
```

# Adding
Adding is pretty straight forward. 

``` 
git add <file>
git add -A # Nice if you have a .gitignore
```

However sometimes you make unrelated changes in the same file. Maybe you add multiple resources to a CFN template but would like to show their addition in two commits because they are not related. For this you need to do an interactive add

```
git add -i tags
```

# Branches

Branches are cheap in git because they are just pointers to commits

```bash
git checkout -b new_branch  # Easiest way to create a new branch and switch to it immediately.
git checkout -b newbranch <commit>
```

## Branches under the hood
Branches are just pointers to commits. You can see this using `git log`
```bash
git branch foo 4b9f3619328b3ec001a671fe2d83b37b7d5ebeed
git log
```

```bash
cd .git/refs/heads # Go to the .git direcory of the repository
ls                   # List branches
cat *                # show commits that branches point to
cp master foobranch  # Make a new branch
git branch -a
```

# Merging
There are two types of merges
 * Fast Forward
 * 3-Way

### Fast forward
A fast forward merge occurs when the common ancestor is the HEAD commit on the target branch. That is, there have not been any commits on the target branch and the comnmit pointer can simply be rolled forward to match the feature branch or the upstream branch (remote branch).

```bash
git checkout -b here-not-hear
perl -pi -e 's/hear/here?/' me
git commit -a -m "hear to here"
git checkout master
git merge here-not-hear
git log
```

### 3 Way Merge
If the branches have diverged, (there have been commits on both branches) then a fast forward is not possible. In this case, the common ancestor and HEADs of both branches are compared, a patch generated, and finally applied in a "merge" commit.

```bash
# Take branch from earlier commit so it appears the branches have diverged
git checkout -b bad-language d9fdacf2d91ba8ed163aec059313ad6165938bd9  
perl -pi -e 's/FUCK/F**K/' issues
git commit -a -m "less bad"
git checkout master
git merge bad-language
git log
```

### Avoiding a Fast-forward Merge
Fast forward merges are really great for integrating upsteam changes into your branches. Some teams frown on this when merging features into a repo's integration branch (often master) because it does not preserve the branching history. If the feature branch is removed it appears like these commits happened on the master branch

```bash
git checkout -b hear-not-here
perl -pi -e 's/here/hear?/' me
git commit -a -m "hear to here"
git checkout master
git merge --no-ff hear-not-here
git log
```

# Rebasing
There's another way to integrate two branches. Rebasing creates a linear history by applying a branch's commits on top of another. It does this by taking and saving commits off of your branch until the common ancestor, applying commits from the other branch, and re-applies your commits.

```bash
# Take branch from earlier commit so it appears the branches have diverged
git checkout -b foo 0255f1b7ef30ddf5c068d5ed9647da3feea90bba
echo "onto" >> rebase
git commit -m "1 commit from the feature branch" rebase
echo "your welcome" >> thanks
git commit -m "2 commit from the feature branch" thanks
git rebase master
```

### When I use merge vs rebase
Personally, I use merge (preferrably PRs) to integrate feature branches into master or development.

I use rebase to integrate upstream/master branch changes into my feature branches (keeping them current). This keeps my changes at the top of the commit history of my branch. Keeping a linear history in the feature branch with my commits on top makes re-writing history easy.

# Fixing things and changing history
```bash
git checkout rebase
git log
git rebase -i 2afae2802708b5545d18c541ba075572682c3cf4
```

```bash
git checkout reset
git log
git reset --soft 2afae2802708b5545d18c541ba075572682c3cf4
```

# Git Aliases
I've included some git aliases in .gitconfig

```
[alias]
        branches = branch -a
        tags = tag
        stashes = stash list
        remotes = remote -v
        commits = log
        lg1 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(bold yellow)%d%C(reset)' --all
        lg2 = log --graph --abbrev-commit --decorate --format=format:'%C(bold blue)%h%C(reset) - %C(bold cyan)%aD%C(reset) %C(bold green)(%ar)%C(reset)%C(bold yellow)%d%C(reset)%n''          %C(white)%s%C(reset) %C(dim white)- %an%C(reset)' --all
        lg = !"git lg1"
```
