#Git #GitMerge #GitRebase 

https://betterprogramming.pub/differences-between-git-merge-and-rebase-and-why-you-should-care-ae41d96237b6

## A comparison of git merge and rebase commands and when to use them

![](https://miro.medium.com/max/700/1*H0368Iggh2NfKP1CK8Pvhw.jpeg)

Photo by [Joey Kyber](https://unsplash.com/@jtkyber1?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/s/photos/traffic?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)

Regardless of which branching strategy your project is using, integrating code changes between branches is something that you need to do almost on a daily basis. With `git` there are two main options for this, either you `merge`, or you `rebase`.

In this article I’ll illustrate and highlight the differences between the two options, and point out things to look out for when performing the actions.

First, I’ll go through the two operations in isolation using animations, finishing off with a side-by-side comparison at the end. If you’re already familiar with how the actions work, feel free to skip to the comparison section.

Reading the official Git manual it states that `rebase` _“reapplies commits on top of another base branch”_, whereas `merge` _“joins two or more development histories together”_. In other words, the key difference between merge and rebase is that while _merge preserves history as it happened, rebase rewrites it_. Before we take a closer look at their respective inner workings to understand what this really means, let’s start with an example.

![](https://miro.medium.com/max/700/1*mdvCMCaHWVBIdn6Xynpc5Q.png)

History tree viewed from the eyes of Satoshi, with remote “origin” aliased as “o” for convenience. Notice also that the local “master” branch (C1) currently trails behind its remote counterpart “o/master” (C4).

Looking at the example above, we see that the developers Ada and Satoshi initially created two topic branches (`feature-1` and `feature-2`), stemming from the same commit (`C1`) on the `master` branch. Ada then completed `feature-1` by merging it into `master` (creating merge commit `C4`). Satoshi now has two options to integrate Ada’s changes into his branch `feature-2` — merge or rebase.

# Merge

Let’s start with the most common workflow for integrating changes: merge. Before Satoshi is ready to merge Ada’s changes into `feature-2`, he must first update his local `master` reference as it’s currently trailing behind. Once `master` and `o/master` are in sync, Satoshi is ready to merge everything into his topic branch.

Checkout this 30 second animation illustrating the process:

Animation illustrating the merge workflow. (Video only, no audio)

With all changes merged into `feature-2` Satoshi can now continue to develop the branch, finishing it off by merging it back into `master` whenever it’s completed.

Below is the final result from the merge action. As you can see, the development history is preserved just as it happened, with only `C7` added.

![](https://miro.medium.com/max/700/1*tqggsi8BEPYPRk5iBqSwuA.png)

By merging the changes from “master” into “feature-2”, history is preserved as it happened and only merge commit “C7” is introduced.

> Confused by the moving HEAD pointer? Checkout my other post on the subject!

[

## What is HEAD in Git?

### HEAD answers the question: Where am I right now in the repository? It is a pointer to the currently checked out branch…

blog.git-init.com



](https://blog.git-init.com/what-is-head-in-git/)

# Rebase

With the basic merge workflow in mind, it’s time to look at the same example but from a rebase perspective. Just as in the merge case, before Satoshi can get going integrating the changes he must first make sure his local and remote `master` branches are in sync. But then, instead of making a regular merge “_preserving history as it happened_”, Satoshi can instead integrate all changes using rebase and hence _“rewrite history”_.

By rebasing `feature-2` onto `master` Git will rewind and reapply the commits `C5` and `C6` one by one, straight onto `C4`, making it look like `feature-2` was originally branched off the tip of Ada’s completed changes.

Checkout this 30 second animation to see the process in action:

Animation illustrating the rebase workflow. (Video only, no audio)

With all changes once again integrated, Satoshi is ready to continue working on his topic branch.

Below is the final result from the rebase action. Notice how commits `C5` and `C6` have been reapplied straight onto `C4`, rewriting the development history and deleting the old commits completely!

![](https://miro.medium.com/max/700/1*V8zg5Hb3QvUiL8u1D5Gu2w.png)

Notice how the hashes for C5 and C6 have changed, simply because it’s actually new commits that have been created (although the content might still be identical).

Now that we know how merge and rebase differ, it’s time to scrutinise the two outcomes in more detail.

# How Merge and Rebase Differ

![](https://miro.medium.com/max/700/1*Y77kjfj3xgPz_YRYa8Zmsg.png)

Side by side illustration of the resulting end states, with the start case on the far left.

As we can see above, the merge operation intertwined the branches together by creating a new single merge commit (`C7`), causing a diamond shaped non-linear history — essentially _preserving history as it happened._ By comparing this result with the outcome from the rebase action we see that no merge commit was created, instead the two commits `C5` and `C6` simply got rewinded and reapplied straight on top of `C4`, keeping the history linear. If we scrutinise the two reapplied commits even further, we can see that the hashes have changed, indicating that rebase truly _rewrites history._

**_Note_**_: Whenever you rebase a branch, new commits will always be generated even though the content might still be the same! So, the former commits will eventually be deleted from history completely._

# With Great Power Comes Great Responsibility

We’ve seen how rebase rewrites history, while merge preserves it. But what does this mean in a broader sense? And what possibilities and potential drawbacks do the two operations come with?

## **Conflicting changes**

Let’s say, for example, you’ve had some nasty conflicts trying to integrate changes. In the merge scenario, you would have only needed to solve the conflicts once, straight in the `C7` commit. With rebase, on the other hand, you could potentially have been forced to solve similar conflicts in each commit (`C5` and `C6`) as they were reapplied.

If conflicts aren’t so straightforward to solve, that may be a sign that you and your colleagues have not been communicating enough as you’ve been working on the same files for too long.

## **Published branches**

Another potential problem related to rebase occurs when the branch you are rebasing has already been published remotely, and someone else has based their work on it. Then, your rebased branch can cause serious confusion and headaches for all involved parties as Git will tell you that your branch is both ahead and behind at the same time. If this happens, pulling remote changes using the rebase flag (`git pull --rebase`) generally solves the problem.

Furthermore, whenever you are rebasing an already published branch, regardless if no one else has based their work on it, you’ll still need to _force push_ it to get your updates to the remote server — overwriting the existing remote reference completely.

## **Loss of data (to your advantage)**

Finally, as rebase rewrites history while merge preserves it, it’s possible to actually lose data when rebasing. When new commits are reapplied, the old ones are (eventually, post garbage collection) deleted. This same trait is in fact what makes rebase so powerful — it allows you to tidy up your development history before making it publicly available! With an _interactive rebase_ (the subject of a forthcoming article) you can for example remove unwanted commits, squash changes together, or simply update commit messages.

# **Rebasing Rules of Thumb**

To avoid the most common issues related to rebase, I’d suggest sticking to the following rules:

-   Don’t rebase a branch that’s been published remotely…
-   …unless you know you are the only one working on it (and feel safe force pushing)
-   Prior to rebasing, create a backup branch from the tip of the branch you’re about to rebase, as it will allow you to easily compare the outcome (once done) and jump back to the pre-rebase state if necessary.

# More Advanced Rebase Cases

There are plenty of more advanced use cases with rebase, which all are beyond scope of this article but will be part of a follow-up article. One of the more prominent features, as already mentioned, is _interactive rebase_ which allows you to customise how each commit should be reapplied.

This mode can be used to split commits, squash commits together, reorder commits, and even delete commits completely, just to name a few possibilities.

# Conclusion

Many developers tend to only use `merge` rather than `rebase`, generally with the comment “At least I know I won’t lose any work.” In a sense, it’s a solid approach to not use tools you aren’t comfortable working with. But not learning to take full advantage of powerful features, even though you know they exist, isn’t such a good approach!

It’s a bit like saying, “I know I have this awesome car but I prefer to stick to the first gear, as I know speed kills,” rather than learning how to shift up and getting comfortable traveling safely at higher velocities.

In my experience, learning how to use `rebase` deepens your understanding of Git in particular, and improves you as an overall developer in general — especially in relation to source code management!

Finally, one of the best pieces of advice I ever got from a senior developer early in my career was this: “Drop the button bashing in Source Tree and learn how to use the Git commands from the terminal instead! Otherwise, you’ll never get the full benefit of using Git, and you won’t be able to script any automation pipelines later on.”

Since then, I’ve only used [GitK](https://git-scm.com/docs/gitk) as a visual aid for inspecting the history tree. I type all commands in the terminal. I suggest you do the same!

Now that you know how merge and rebase differ I hope you feel more confident in starting to use both of them.

Thanks for reading and good luck improving your source code management skills!

Thanks to Anupam Chugh