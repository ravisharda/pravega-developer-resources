# Git Commands and Tasks

## Common Tasks

Assumptions:
* Remote `upstream` is pravega/pravega.git
* Remote `origin` is youracount/pravega.git

## Downloading a newly created branch on your fork

**Pravega:**

```
git clone https://github.com/ravisharda/pravega.git
cd pravega
git branch
git checkout -b <branch-name>
git branch
```

**Pravega Samples:**
Note: In Samples, PRs are against develop branch, not against master (please, check https://github.com/pravega/pravega-samples/wiki/How-to-release#about-this-repository). 

```
git clone https://github.com/ravisharda/pravega-samples.git
cd pravega-samples
git branch
git checkout -b issue-181-security-samples
git branch
```

## Merge commits from main repo master into your fork's master (Syncing a Fork)

|S.No.|Command|Description|Verification (if any)|
|:---:|:------|:----------|- |
|1|``git clone https://github.com/ravisharda/pravega.git``|Download the forked copy from Github|- |
|2|``cd pravega``|-|- |
|3|``git remote add upstream https://github.com/pravega/pravega.git``|Add an upstream remote branch, which will facilitate pulling down new commits from the main repo|``git remote -v`` or ``git remote show upstream``|
|4|``git fetch upstream``|Fetch the branches and their respective commits from the upstream repository. Commits to master will be stored in a local branch, upstream/master.|-|
|5|``git checkout master``|Check out your fork's local master branch|``git branch``|
|6|``git merge upstream/master``|Merge the changes from upstream/master into your local master branch. This brings your fork's master branch into sync with the upstream repository, without losing your local changes.|-|
|7|``git push origin master``|Syncing your fork only updates your local copy of the repository. To update your fork on GitHub, you must push your changes.|-|
|8| Go to ``https://github.com/ravisharda/pravega/`` and inspect the message|Verify that your fork's master is even with upstream.|

Source: [GitHub Help - Syncing a Fork](https://help.github.com/articles/syncing-a-fork/#platform-windows)

### Rebasing a branch on your fork to master of your fork

```
git clone https://github.com/<youraccount>/pravega.git
cd pravega
git checkout <branch-name>
git rebase master
git log
git push origin <branch-name>
```

## Pulling all changes 

* Pulling all changes...
  * ``git pull``
  * ... in the remote repo to master: ``git pull origin master``
  * ... in the remote repo to a branch: ``git pull origin <branch-name>``

(Will fail if there are merge conflicts)

## Creating a new branch from command line 

```
git clone https://github.com/ravisharda/pravega.git
git checkout -b issue-3227-auth-logic
(verify using 'git branch')
git push origin issue-3227-auth-logic
```

## Merging a PR to a branch to test out the changes

```
# git fetch origin pull/ID/head:BRANCHNAME
# Assumptions:
# PR no. = 3356, branch that you want to test out is: r0.4
#
git clone https://github.com/pravega/pravega.git
git fetch origin pull/3356/head:r0.4
git checkout r0.4
```
See more at https://help.github.com/en/articles/checking-out-pull-requests-locally

Similar experience, when trying out a colleague's PR ([PR 217](https://github.com/pravega/flink-connectors/pull/217) `vijikarthi:issue-pravega-security`):
```
Forked the repo

git clone https://github.com/ravisharda/flink-connectors.git

git remote add upstream https://github.com/pravega/flink-connectors.git

git fetch upstream pull/217/head:issue-pravega-security
```

Yet another way - actually a much simpler one: 
```
# Recursive is necessary for flink-connections project
git clone --recursive -b issue-pravega-security https://github.com/vijikarthi/flink-connectors
```
## Updating an Exisiting Branch on Your Fork with Latest Code From Master on Main Repo

1. Download the branch from your fork: ``git clone -b <branch_name> https://github.com/ravisharda/pravega.git``
2. Add an upstream remote branch, which will facilitate pulling down new commits from the main repo: ``git remote add upstream https://github.com/pravega/pravega.git``. (Verify using ``git remote -v``)
3. Perform the rebase: 
   1. First make sure you are currently in the feature branch: ``git branch``
   2. Fetch all the new changes from the main repo/upstream: ``git pull --rebase upstream master``
   3. Fix any mrge conflicts that might arise. 
   4. Continue rebasing: ``git rebase --continue``
   5. Verify the code. 
   6. Now, push all the changes you have made to the branch to the master of your forked repo: ``git push --force origin <pull-request-branch-name>``  

## Updating a New Branch on Your Fork with Latest Code From Master on Main Repo
1. Download the forked copy from GitHub: ``git clone https://github.com/ravisharda/pravega.git``
2. Add an upstream remote branch, which will facilitate pulling down new commits from the main repo: ``git remote add upstream https://github.com/pravega/pravega.git``. (Verify using ``git remote -v``)
3. Make a new branch: ``git checkout -b <branch_name>``
4. Make changes to the code as needed. 
5. Commit the changes you have made. 
6. Push up to your GitHub fork: 
   1. First make sure you are currently in the feature branch: ``git branch``
   2. Fetch all the new changes from the main repo/upstream: ``git pull --rebase upstream master``
   3. Fix any mrge conflicts that might arise. 
   4. Continue rebasing: ``git rebase --continue``
   5. Verify the code. 
   6. Now, push all the changes you have made to the branch to the master of your forked repo: ``git push --force origin <pull-request-branch-name>``  
   
In this case your forked repo/branch will be ahead of forked repo/master. Not something you often want to do. It might be useful when you want to forked branch to track a branch on the main repo. 

## Targeting a PR to a branch other than Master

I used these steps for one of the PRs in pravega-samples repo:

1. `git clone https://github.com/ravisharda/pravega.git`
2. Creates issue-181-security-samples-dev branch off develop: `git checkout -b issue-181-security-samples-dev develop`
3. Checkout the new branch: `git checkout issue-181-security-samples`
4. Made changes. 
5. Committed and pushed as usual. 

## Undoing changes made to fork's master by mistake

Use git reset approach described [here](https://www.atlassian.com/git/tutorials/undoing-changes).
1. Identify the commit you want to revert to: `git log --oneline`
2. `git reset --hard d9a15c2b6`
3. `git push --force`

## Reverting your last change
git reset HEAD~1 // this undo the last commit
//git add -u (optionally, add some changes)
git commit -s 
git push --force
