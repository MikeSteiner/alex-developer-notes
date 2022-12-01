#Git  #GitMerge #GitRebase 

In Git, there are two main ways to integrate changes from one branch into another: the `merge` and the `rebase`.

For example, there are two branches:
- ***main***  
- ***feature*** 

### Merge

We have to be inside the main branch
```
git merge feature
```
This command will move all the commits from the feature branch and include them in the history of the master/main. On top of the latest commit in the main.

```
git merge --squash feature
```
Squash option summarize all the commits from the feature branch in one single commit.  The history of the feature branch will be lost, but on the other hand will not pollute the main branch.

### Rebase
[edit]