#### init
This command is used to create a new repository for projects. It can also be used to convert an existing project to a new repository or an empty repository.

```
git init [repository name]
```

---
#### Config

This command is a function that is used to set configuration values on a global or local level. It can be used to set the author name and email ID with commit.
**Usage:**

```
git config -global user.name "[name]"

git config -global user.email "[email]"
```

---
#### Clone
This command is used to create a copy of an existing repo in a new directory at a different location. Users majorly use it to copy a repository into the local system. This is the key command that gives the distribution feature to GIT. 

```
git clone [repo url]
```

---
#### Add
This command adds the modified files into the staging area. It shows GIT that you wish to include particular modifications or files in the next commit. Although it is seen as an unnecessary step, it allows the creation of history without changing the style of working. It is a simple, important, and powerful tool. 

```
git add [filename]

git add .
```

> The filename can be replaced with a * to add all the modified files to the staging area at once.

---
#### Commit

The git commit command is the primary and core command of git. It is used to create a timeline of the staged changes along the timeline of the project. Committing allows accumulating major changes made by the user on a local level before pushing it to the global level. 

```
git commit -m "Your Commit Message"
```
---
#### Difference

The diff command displays the differences in files between two commits or between a commit or after the commits are done. Changes including lines added, removed, or modified can be seen using this command. 

```
git diff
```

---
#### Log

The log gives a history of commits. This command shows the list of all the commits that have been made to the repository. The hash, the commit message, and all the metadata related to the commit can be seen upon using this command. 

```
git log
```

---
#### Reset

git reset is a command used to undo the committed changes from a repository. It undoes the work without changing the file contents. For instance, if you add or commit two files instead of one, you can remove that unnecessary file through this command.

```
git reset
```

---
#### Status

The status command displays the current status of the repository such as untracked files, added files, changed files, etc.

```
git status
```

---
#### Remove

This command is used to remove or delete a file from the working directory. Fortunately, the remove command not only removes a file from the local directory but also adds the changes into the staging area, thus avoiding the usage of add. 

```
git rm [filename]
```

---
#### Tags

Tags are used to tag specific points in the history and used to reference them in a future version of the project. For instance, if you feel that a major commit in your project is important, you can tag it using the command. This gives your project a version tag that can be accessed when necessary. 

```
git tag [commit ID]
```

---
#### Show
The show command is used to view the expanded details about the tags, trees, commits, etc. It gives a response based on the object called. 

```
git show [commit]
```

---
#### Branch

Branching is one of the strongest features of git. It is used to make a checkpoint and create a new feature without disturbing the actual project. If the branched feature proves to be effective, it can be merged with the main project. There are several commands within the branch command. 

-   **git branch**: It is used to show the current branch
-   **git branch [branch]**: It is used to create a new branch. 
-   **git branch -d [branch]**: It is used to delete a branch. 
-   **git checkout [branch]**: It is used to switch between branches.
-   **git checkout -b [branch]**: It is used to create a new branch and switch to it. 

---
#### Merge

The merge feature allows the merging of two distinct branches into one single branch. However, there might be merge conflicts if more than one person merges at the same time without pulling each other’s work. Conflicts can be resolved separately. 

```
git merge [branch name]
```

---
#### Remote

Remote command is very useful as it helps in connecting a local repository to a cloud-based repository. This allows collaboration of the project from anywhere in the world. 

```
git remote add origin [server link]
```

---
#### Push

The command is used to push all the local staged changes into a remote repository stored in the cloud.

```
git push origin [branch name]
```

---
#### Pull

The pull command fetches and merges the changes from the remote repository in a cloud into the local system. 

```
git pull
```

---
https://www.interviewbit.com/blog/git-commands/
https://education.github.com/git-cheat-sheet-education.pdf