## Cheatsheet collection

* [Home](https://dejanu.github.io/)

### Git

* Git has 3 levels of configuration:
  * SYSTEM(configurations for multiple users)
  * GLOBAL(configurations used for one user all repo on one host) 
  * LOCAL (configuration for one repo) 
  
```bash
#list the configuration
git config --list 

#edit global configuration 
git config –e  --global

# get/edd the user name
git config user.name
git config --global user.name "NAME"

```
  
* Primitive objects:
  * `commit` — object which contains the reference to another tree object
  * `tree` — object which contains references to other blobs or subtrees.
  * `blob` — object used for storing the contents of a single file.
  ```bash
  git add => BLOB
  git commit ==> COMMIT + TREE
  ```
* HEAD = pointer to the last commit on the branch
* `git pull` == `git fetch` + `git merge`
* `git checkout -b` = `git branch` + `git checkout`

## Configure Git

```bash

$ git config --global user.name <YOUR NAME>
$ git config --global user.email <YOUR MAIL>

#disable SSL verification if you encounter SSL certificate problem: self signed certificate
$ git config --global http.sslVerify false
```

***

## Commands

```bash
# view/add remote
git remote -v
git remote add origin remote_repository_URL
git remote set-url origin git://new.url.  //change the url repo
git remote remove origin


#  shorthand for git checkout -b [branch] [remotename]/[branch]
git checkout --track origin/remote_branch
git fetch <remote> <rbranch>:<lbranch> 
git checkout <lbranch>

# delete local/remote branch
git branch -D local_branch
git push --delete origin remote_branch

# clean working directory interactive mode
git clean -i

# UNDO any changes in the WORKING AREA , remove file from git tracking
git checkout -f <FILE> 

# Reset current HEAD to the specified state (MIXED the default one)
git-reset

# UNDO last commit and preserve changes
git reset --soft HEAD^ == git reset --soft HEAD~1 

# UNDO last commit and remove changes, combine both git reset and git checkout in a single command
git reset --hard HEAD~1 

# much wow (undo 2 commits) 
git reset --head X 2 == git reset HEAD^^ 

# create a new commit which usually has the inverse effect of the commit being reverted.
git revert HEAD --no-edit | git revert HASH

```


***

## FunStuff

```bash
# delete all branches merged into develop
git branch -r --merged develop | grep -vE "develop|master|release" | xargs -n 1 git branch -d

```
 

***

## Inspection


```bash

# check commits 
git log --author="Jon"
git log --graph --oneline --all

# examine the contents of a file
git blame <FILE>
gitk --follow <FILE>

# modify last commit
git commit --amend 

# list remote branches
git branch -va

# check commit
git show COMMIT_HASH

```