## Cheatsheet collection

* [Home](index.md)
* [Ansible](ansible.md)
* <ins>[Git](git.md)</ins>
* [GCP](gcp.md)
* [Docker](docker.md)
* [Azure](azure.md)
* [Terraform](terraform.md)
* [Helm](helm.md)
* [ElasticSearch](elastic.md)
* [Kubernetes](k8s.md)
* [Istio](istio.md)
* [OIDC](openID.md)
* [PostgreSQL](postgres.md)
* [GitHub Copilot](copilot.md)

---

### Nice stuff

* [Manual Page](https://mirrors.edge.kernel.org/pub/software/scm/git/docs/)
* [D3 visual](http://onlywei.github.io/explain-git-with-d3/)
* [oshitgit](https://ohshitgit.com/)
* [101](https://www.git-tower.com/learn/git/faq/restoring-deleted-files)

  
* Primitive objects:
  * `commit` — object which contains the reference to another tree object
  * `tree` — object which contains references to other blobs or subtrees.
  * `blob` — object used for storing the contents of a single file.
  ```bash
  git add => BLOB
  git commit ==> COMMIT + TREE
  ```
* `HEAD` = pointer to the last commit on the branch
* `git pull` == `git fetch` + `git merge`
* `git checkout -b` = `git branch` + `git checkout`

## Configure Git

* Git has 3 levels of **configuration**:

**SYSTEM** (configurations for multiple users)
**GLOBAL** (configurations used for one user all repo on one host)
**LOCAL** (configuration for one repo)

```bash
# list the configuration: cat ~/.gitconfig 😉
git config --list 

# edit global configuration 
git config –e --global

# get/add the user name
git config user.name
git config --global user.name "YOUR_NAME"
git config --global user.email "MAIL"

# Disable SSL verification if you encounter SSL certificate problem: self signed certificate
git config --global http.sslVerify false
```
***

## SSH stuff

```bash
# Use other ssh key
git -c core.sshCommand="ssh -i ~/.ssh/<PRIVATE_KEY>" clone git@github.com:dejanu/sretoolkit.git

# Update ~/.ssh/config

Host github.com
    Hostname github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/<private_key>
```
***
## Commands

* `checkout` and `reset` are generally used for making **local or private undos**, they'll modify the history of a repo and can cause conflicts when pushing to remote
* `revert` is considered a safer operation for **public undos** as it creates new history which can be shared remotely and doesn't overwrite history remote

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

# UNDO changes in working directory a.k.a I've deleted a file and did not commit
git checkout -f <FILE>
git reset HEAD <filename>

# UNDO (unstage) changes in staging area 
git restore --staged <file>

# Reset current HEAD to the specified state (MIXED the default one)
git reset

# UNDO last commit and PRESERVE changes
git reset --soft HEAD^ == git reset --soft HEAD~1
# much wow (undo 2 commits) 
git reset --head X 2 == git reset HEAD^^ 

# UNDO last commit and REMOVE changes, combine both git reset and git checkout in a single command
git reset --hard HEAD~1 

# create a new commit which usually has the inverse effect of the commit being reverted.
git revert HEAD --no-edit | git revert HASH

# rename local branch, push it and delete the old branch
git branch -m <oldname> <newname>
git push origin -u <newname>
git push origin --delete <oldname>

# rebase: rewrite branch's history by moving or combining a sequence of commits to a new base commit
# SQUASH:  interactive rebase, allows you to edit, squash, last 2 commits
git rebase -i HEAD^^ 
## pick commit_hash FIRST_COMMIT
## squash commit_hash SECOND_COMMIT 
```

***

## FunStuff
[GitHooks article](https://dev.to/dejanualex/short-intro-to-git-hooks-2llb)
```bash
# before counting unmerger/stale branches
git remote update origin --prune

# delete all branches merged into develop from local and remote
git branch -r --merged develop | grep -vE "develop|master|release" | xargs -n 1 git branch -d
git branch -r --merged master | grep -vE "develop|master|release" | awk -F'/' '{print $2}'|xargs -n 1 git push --delete origin

# diff MERGED vs UNMERGED branches
git branch -r --merged | grep -v HEAD | xargs -L1 git --no-pager log --pretty=tformat:'%Cgreen%d%Creset | %h | %an | %Cblue%ar%Creset' -1 | column -t -s '|'
git branch -r --no-merged | grep -v HEAD | xargs -L1 git --no-pager log --pretty=tformat:'%Cgreen%d%Creset | %h | %an | %Cblue%ar%Creset' -1 | column -t -s '|'
```
```bash
# loop through each unmerged branch and tell you (a) when the last commit was made, and (b) how many commits it contains which are not merged to ‘origin/master’
for b in $(git branch --remote --no-merged); do

    echo $b;

    git show $b --pretty="format:  Last commit: %cd" | head -n 1;

    echo -n "  Commits from 'master': ";

    git log --oneline $(git merge-base $b origin/master)..$b | wc -l;

    echo;

done
```
```bash
# pull all remote branches
for remote in `git branch -r`; do git branch --track ${remote#origin/} $remote; done
```

```bash

# check the parent branch

git show-branch | grep '*' | grep -v \"$(git rev-parse --abbrev-ref HEAD)\" | head -n1 | sed 's/.*\\[\\(.*\\)\\].*/\\1/' | sed 's/[\\^~].*//' #
git log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
git log --first-parent
```

***

## Inspection


```bash

# check commits 
git log --author="Jon"
git log --graph --oneline --all

# check only the commit msg aka subject
git log --pretty=format:"%s"

# examine the contents of a file
git blame <FILE>
gitk --follow <FILE>

# modify last commit
git commit --amend 

# list remote branches
git branch -va

# check remote unmerged branches
git branch -r --no-merged

# check commit
git show COMMIT_HASH

```
---

```bash
                    ___ _____
                   /\ (_)    \
                  /  \      (_,
                 _)  _\   _    \
                /   (_)\_( )____\
                \_     /    _  _/
                  ) /\/  _ (o)(
                  \ \_) (o)   /
                   \/________/         @dejanualex
```