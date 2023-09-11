If you are new to GIT, I like to describe it as a rotary selector for your file system. At any point in time, you can tell GIT to point to a branch, which holds the files at a certain point in time. One of the advantages of this system is to allow changes/fixes/enhancements to be made within a branch while not affecting the rest of the environment. Once tested the branch can either be merged into the dev branch or deleted. Be aware however, uncommitted changes within a branch will follow across switching branches; so it is best to commit any changes before switching.

## The Basics
#### List branches
* Consider lsb [alias](#Aliases)
1. Get current changes
   * `git fetch -p`
1. `git branch -av`

#### Switch branch
1. [Commit changes](#Commiting changes)
1. [List Branches](#List branches)
1. `git switch <branch>`
1. `git pull -p`

#### Creating a new branch:
* Consider mkb [alias](#Aliases)
1. [Commit changes](#Commiting changes)
1. Get current changes
   * `git fetch -p`
1. Create branch
   * `git checkout --no-track -b <bug|task-ID-useful_description_to_others> origin/dev`
   * `git checkout --no-track -b hotfix-{id}-* origin/main`
1. Create branch on GitLab and set it to track
   * `git push -u origin HEAD`

#### Adding new files
1. Get list of untracked files
   * `git status`
1. Add files
   * `git add <file1 | regex> <...>`
   * `git add .`
1. Verify you only added what you thought you were adding
   * `git status`
1. [Commit changes](#Committing changes)

#### Committing changes
1. `git status`
1. `git commit -m "Meaningful message about the changes included in this commit" <options>`
   * -a : include all modified tracked files
   * <file | regex>... : only the listed files in this commit
1. Optional, but at least at the end of the day: Push your changes to GitLab
   * `git push`

#### Collaborating
1. Primary developer push changes up to gitlab
   * `git push`
1. Secondary developer switches to the branch for the first time changes into their copy of the branch
   1. [List branches](#List branches)
   1. [Switch to branch](#Switch branch)
1. Secondary developer makes changes
   1. [Commit changes](#Commiting changes)
   1. `git pull -p`
   1. [Resolve merge conflicts](#Merge conflict resolution)
   1. `git push`
1. Primary developer pulls secondaries changes
   1. [Commit changes](#Commiting changes)
   1. `git pull -p`
   1. [Resolve merge conflicts](#Merge conflict resolution)
   1. `git push`

#### Delete local copies of branches
* `git branch -d <branch>`
* `git branch -D <branch>`

#### Rename branch
1. [Commit changes](#Commiting changes)
1. [Switch to branch](#Switch branch)
1. Delete branch from origin
   * `git push origin :$(git symbolic-ref --short HEAD)`
1. Rename local copy of branch
   * `git branch -m <new name>`
1. `git push -u origin HEAD`

## Advanced topics
#### Aliases
Aliases allow for custom commands to be created. These can be a git command with a set of flags, or by using an '!' at the beginning a shell command to accomplish a set of related tasks. Alias commands run just like any other git command `git <cmd> [params]` so `git <alias> {params]`
* **alias** - description - what it does
   * `alias command`
* **lsa** - Print all the aliases
   * `git config --global alias.lsa 'config --get-regexp ^alias'`
* **lsb** - List all existing branches. It performs a fetch then list all remote branches.
   * `git config --global alias.lsb '!git fetch -pq;git for-each-ref --sort=committerdate refs/heads refs/remotes --format="%(align:width=13)%(upstream:track)%(end) %(align:width=27)(%(color:green)%(committerdate:relative)%(color:reset))%(end) %(align:width=25)%(authorname)%(end) %(color:yellow)%(refname:short)%(color:reset)"'`
* **mkb** - Make [new] branch and switches the user to the new branch - Checks that there is nothing needing to be committed, fetches from origin to get all possible source branches, prompts the user to select a source branch, prompts the user to select a branch type, prompts the user for the redmine ID, and prompts for a description; it formats the name to <type>-<id>-desc_rip_tion> format, and provides the user a chance to correct the formatted name; once the name is verified, the branch is created from the selected source branch and sets up the tracking branch on origin before switching to the new branch.
   * `git config --global alias.mkb '!git diff-index --quiet HEAD -- && (git fetch -q;PS3="Select source branch: ";select S in $(git branch -r --format "%(refname:short)");do break;done;if [ ! -n "${S}" ];then exit;fi;PS3="Select branch type: ";select A in task bug;do break;done;if [ ! -n "${A}" ];then exit;fi;read -p "Enter redmine id: " B;if [ ! -n "${B}" ];then exit;fi;read -p "Description: " C;if [ ! -n "${C}" ];then exit;fi;read -ep "Confirm: " -i "${A}-${B}-${C// /_}";if [ ! "${REPLY}" ];then exit;fi;git fetch -p;git checkout --no-track -b ${REPLY// /_} ${S};git push -u origin HEAD) || echo "Aborted: Uncommitted changes"'`
* **sync** - Sync branch with origin/dev - Checks that there is nothing needing to be committed, fetches origin, merges orgin/dev into the current branch, if successful pushes the merge.
   * `git config --global alias.sync '!git diff-index --quiet HEAD -- && (git fetch -p;PS3="Select source branch: ";select S in $(git branch -r --format "%(refname:short)");do break;done;if [ ! -n "${S}" ];then exit;fi;git merge ${S} && git push || echo "Not pushed due to merge conflict") || echo "Aborted: Uncommitted changes"'`
* **lg** - Print log entry with files modified in commit and brief SHA for cherry-pick.
   * `git config --global alias.lg "log --color --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --name-status --"`

#### Stashing
Stashing is the process of saving your work locally without committing them to the repository so that you can temporarily switch to a separate task or apply a set of changes multiple times.
* List all stashes
   * `git stash list`
* Creating a stash
   * `git stash`
   * `git stash -u` # include untracked (new) files
* Delete stash
   * `git stash drop stash@{1}`
   * `git stash clear` # all
* Apply the changes from the stash, while keeping the stash.
   * `git stash apply` # stash@{0}
   * `git stash apply stash@{1}`
* Apply the changes from the stash, removing the stash.
   * `git stash pop` # stash@{0}
   * `git stash pop stash@{1}`
* Show diff against stash
   * `git stash show [-p] stash@{2}` # -p = full dff
* Create branch from stash
   * `git stash branch <bug-{ID}|task-{ID}>-<useful_description_to_others> stash@{1}`
#### Syncing between multiple local repository clones
temp:
```text
~/repo1 $ git remote add repo2 ~/repo2
~/repo1 $ git fetch repo2
~/repo1 $ git merge repo2/foo
```
#### Merge conflict resolution
[Resolve merge conflicts](https://www.google.com/search?q=git+merge+conflict+resolution)
* What is a merge conflict?
   A conflict occurs when the file you are merging has been modified on both remote and local in the same place and GIT cannot determine which code it should keep.
* Manual resolution (Do not add your code changes to the merge commit)
   1. Use `git status` to determine which files are in conflict
   1. Find the conflict markers (<<<<<|=====|>>>>>) in the conflicted file
   1. Determine which code needs to remain
      * Sometimes this is their code
      * Sometimes this is your code
      * Sometimes you need to take parts from both
   1. Stage the file in git `git add <FILE>`
   1. Repeat from 1 until all conflicts are resolved
   1. Complete the merge with a `git commit` without any other options
* Use the merge tool `git mergetool`
   1. Setup the merge tool
* Use ATOM which is GIT aware (GIT library is installed by default)
   1. Open conflicted file in ATOM
