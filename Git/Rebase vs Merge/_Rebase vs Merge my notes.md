#Git  #GitMerge #GitRebase 

https://www.youtube.com/watch?v=dO9BtPDIHJ8     
#Rate10
https://www.youtube.com/watch?v=CRlGDDprdOQ
#Rate7

### Starting point

In Git, there are two main ways to integrate changes from one branch into another: the `merge` and the `rebase`.

The starting point is presented in the figure below, where you diverged your work and made commits on two different branches.

![[Pasted image 20221202114147.png]]

### Merge

We have to be inside the main branch
```
git merge experiment
```
This command will move all the commits from the feature/experiment branch and include them in the history of the master/main. On top of the latest commit in the main.

```
git merge --squash experiment
```
Squash option summarize all the commits from the feature/experiment branch in one single commit.  The history of the feature branch will be lost, but on the other hand will not pollute the main branch.

![[Pasted image 20221202114250.png]]

### Rebase

There is another way: you can take the patch of the change that was introduced in `C4` and reapply it on top of `C3`. In Git, this is called _rebasing_. With the `rebase` command, you can take all the changes that were committed on one branch and replay them on a different branch.

For this example, you would check out the `experiment` branch, and then rebase it onto the `master` branch as follows:

```console
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: added staged command
```

This operation works by going to the common ancestor of the two branches (the one you’re on and the one you’re rebasing onto), getting the diff introduced by each commit of the branch you’re on, saving those diffs to temporary files, resetting the current branch to the same commit as the branch you are rebasing onto, and finally applying each change in turn.

![Rebasing the change introduced in `C4` onto `C3`](https://git-scm.com/book/en/v2/images/basic-rebase-3.png)

Figure 37. Rebasing the change introduced in `C4` onto `C3`

At this point, you can go back to the `master` branch and do a fast-forward merge.

```console
$ git checkout master
$ git merge experiment
```

![Fast-forwarding the `master` branch](https://git-scm.com/book/en/v2/images/basic-rebase-4.png)

Figure 38. Fast-forwarding the `master` branch

Now, the snapshot pointed to by `C4'` is exactly the same as the one that was pointed to by `C5` in [the merge example](https://git-scm.com/book/en/v2/ch00/rebasing-merging-example). There is no difference in the end product of the integration, but rebasing makes for a cleaner history. If you examine the log of a rebased branch, it looks like a linear history: it appears that all the work happened in series, even when it originally happened in parallel.

Often, you’ll do this to make sure your commits apply cleanly on a remote branch — perhaps in a project to which you’re trying to contribute but that you don’t maintain. In this case, you’d do your work in a branch and then rebase your work onto `origin/master` when you were ready to submit your patches to the main project. That way, the maintainer doesn’t have to do any integration work — just a fast-forward or a clean apply.

Note that the snapshot pointed to by the final commit you end up with, whether it’s the last of the rebased commits for a rebase or the final merge commit after a merge, is the same snapshot — it’s only the history that is different. Rebasing replays changes from one line of work onto another in the order they were introduced, whereas merging takes the endpoints and merges them together.