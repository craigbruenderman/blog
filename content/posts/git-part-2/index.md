---
title: "Git Primer (Part 2)"
date: "2025-07-29"
series: ["Git"]
series_order: 1
categories: ["Devops", "Cloud"]
tags: ["Git"]
---

## Git Branches

Continuing on from the first post, branches are a key concept in Git and they can be thought of as an extension to commits. Just by virtue of creating the repo, I already have a branch which is the primary line of development. This default branch is typically called __main__ or __master__, depending on your version of Git and is configurable in Git from version 2.28.0 onwards; mine is set to **master**.

As commits accumulate in the default main/master branch, they accumulate like so:

{{< mermaid >}}
---
config:
  theme: 'base'
  gitGraph:
    mainBranchName: "master"
---
gitGraph
  commit
  commit
  commit
{{< /mermaid >}}

Branches are nothing but containers which contain commits. Assume we need our repo to diverge in some way from the main line of contribution so that we can make changes in a particular direction without affecting the primary line of development. An example is needing a specific bug fix that we want to work on independently, without painting ourselves in a corner where those changes cannot be incorporated into the main line of development later. We also want to leave the main branch in a state that it can continue to be used and deployed without worrying about what is going on with that bug fix.

Now I'll create an additional branch to veer off into some other line of development. By running `bit branch <branch-name>`, I create a new branch which uses the most recent commit as its starting point.

![](/images/git-branch.png)

{{< alert icon="lightbulb" >}}
Notice that after creating **test-branch**, I checked it out. This context switch within the repo is indicated by the green prompt which now shows **new-branch** instead of **master**.
{{< /alert >}}

{{< mermaid >}}
---
config:
  gitGraph:
    mainBranchName: "master"
---
gitGraph
  commit
  commit
  commit
  branch bugfix
  commit
  commit
  checkout master
  commit
  merge bugfix
{{< /mermaid >}}

The diagram indicates that after the 3rd commit in **master**, a new branch called **bugfix** was created. 2 commits were made in the **bugfix** branch to fix the bug, while 1 additional commit was made to **master**. Finally, the bugfix branch was **merged** back into **master** so that **master** could incorporate the changes made in the *bugfix** branch.

### Git Checkout

In addition to creating and checking out a branch in separate steps, it can be done in a single step by `git checkout -b <branch name>`. So now we've seen that `git checkout` is used to switch between branches, but it can also be used to switch to a certain snapshot (commit).

![](/images/git-checkouts.png)

Notice the directory listing shows that the latest commit in the **master** branch includes 2 text files. Then I created a new branch, added and committed a file to it, so that **bugfix** has 3 files. Then, switching back to **master**, that commit still only has 2 files.

![](/images/git-log-summary.png)

`git log --summary` indicates the file creation relative to each branch. The **bugfix** branch contains all 3 files, since it inherited 2 of the files when branched from **master**, and then got a third file of its own.

### Merging

When a branch is ready to be merged, issue `git merge <branch to merge>` from within the context of the branch you want to merge into.

![](/images/git-merge.png)

## Summary of Commands

| Command | Function |
| :------- | :------: |
| `git init`  | One time initialization of a repo |
| `git add <file(s)>`| Stage changes for teh next commit |
| `git commit -m "comment"` | Create a commit for staged changes including a comment |
| `git status` | Get current repo status of staged changes |
| `git log` | Print order list of commits " |
| `git checkout <id>` | Temporarily move back to commit <id> |
| `git revert <id>` | Revert the changes of commit <id> by creating a new commit |
| `git reset <id>` | Undo commit(s) up to commit <id> by deleting commits |

## Conclusion

That wraps up the whirlwind primer for Git. I've skipped a lot, so head to that book for much better coverage. This post was mainly helpful for me to dust off some Git cobwebs and play with some features of Hugo+Blowfish.