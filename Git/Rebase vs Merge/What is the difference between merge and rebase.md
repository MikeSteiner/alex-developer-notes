#Git #GitMerge #GitRebase 

https://stackoverflow.com/questions/16666089/whats-the-difference-between-git-merge-and-git-rebase

Reading the official Git manual it states that **“rebase reapplies commits on top of another base branch”**, whereas **“merge joins two or more development histories together”**. In other words, the key difference between merge and rebase is that while `merge` preserves history as it happened, `rebase` rewrites it.

Let's contextualize these statements with a side-by-side example!

[![Comparison of merge vs rebase](https://i.stack.imgur.com/gPlXA.png)](https://i.stack.imgur.com/gPlXA.png)

As illustrated above, the `merge` operation intertwined the branches together by creating a new single merge commit (**C7**), causing a diamond shaped non-linear history — essentially preserving history as it happened. By comparing this result with the outcome from the `rebase` action we see that no merge commit was created, instead the two commits **C5** and **C6** simply got rewinded and reapplied straight on top of **C4**, keeping the history linear.

If we scrutinise the two reapplied commits even further, we can see that the hashes have changed, indicating that `rebase` truly rewrites history.

#### Worth noting

Whenever you `rebase` a branch, new commits will always be generated even though the content might still be the same! That said, any former commits will eventually (post garbage collection) be deleted from history if no other pointer (branch/tag) is referencing them.

#### With great power comes great responsibility

We’ve seen how rebase rewrites history, while merge preserves it. But what does this mean in a broader sense? And what possibilities and potential drawbacks do the two operations come with?

**Conflicting changes**

Let’s say, for example, you’ve had some nasty conflicts trying to integrate the changes. In the merge scenario, you would have only needed to solve the conflicts once, straight in the **C7** commit. With rebase, on the other hand, you could potentially have been forced to solve similar conflicts in each commit (**C5** and **C6**) as they were reapplied.

**Published branches**

Another potential problem related to rebase occurs when the branch you are rebasing has already been published remotely, and someone else has based their work on it. Then, your rebased branch can cause serious confusion and headaches for all involved parties as Git will tell you that your branch is both ahead and behind at the same time. If this happens, pulling remote changes using the rebase flag (git pull --rebase) generally solves the problem.

Furthermore, whenever you are rebasing an already published branch, regardless if no one else has based their work on it, you’ll still need to force push it to get your updates to the remote server — overwriting the existing remote reference completely.

**Loss of data (to your advantage)**

Finally, as rebase rewrites history while merge preserves it, it’s possible to actually lose data when rebasing. When new commits are reapplied, the old ones are (eventually, post garbage collection) deleted. This same trait is in fact what makes rebase so powerful — it allows you to tidy up your development history before making it publicly available!

#### Conclusion

While `merge` is safe to use from a potential data loss perspective, and can feel more straight forward to use. Here are some pointers that can help you avoid the most common issues related to `rebase`.

-   Don't rebase a branch that’s been published remotely…
-   …unless you know you are the only one working on it (and feel safe force pushing)
-   Prior to rebasing, create a backup branch from the tip of the branch you’re about to rebase, as it will allow you to easily compare the outcome (once done) and jump back to the pre-rebase state if necessary.

_**Source:**_ Above excerpt is taken from this full length post on the subject: [Differences Between Git Merge and Rebase — and Why You Should Care](https://betterprogramming.pub/differences-between-git-merge-and-rebase-and-why-you-should-care-ae41d96237b6)