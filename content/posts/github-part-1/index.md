---
title: "Git Primer"
draft: true
date: "2025-07-26"
series: ["Github"]
series_order: 1
categories: ["Devops", "Cloud"]
tags: ["AWS", "Github", "CI/CD"]
---

{{< typeit 
  tag=h4
  speed=50
  loop=false
>}}
git init
git add .
git commit -m "First commit!"
{{< /typeit >}}

## Series Overview

In this series, I'll be looking at the Git version control system, along with Github's basic function as a remote repository, and Github's feature called Github Actions as part of Continuous Integration / Continuous Delivery. I am certainly not a professional software engineer, but these are just a few of the tools I incorporate for the types of development work I do.

In fact, this very blog post you're reading right now is brought to you by a relatively simple use of Git, Github, and Github Actions along with the Hugo static website generator and a theme called Blowfish. I'll cover the workflow later as an example of some fundamentals.

If you want way more depth than I can offer, [this free book](https://git-scm.com/book/en/v2/Getting-Started-About-Version-Control) is excellent.

## Git Overview

Git is a popular tool for [distributed version control](https://en.wikipedia.org/wiki/Distributed_version_control) that was built by Linus Torvalds over 20 years ago to manage the large distributed development cycle for the Linux kernel. Git can be used to maintain a file repository for a single user on a single local machine, or by many collaborators working on multiple versions of multiple projects with mechanisms to synchronize the development work. Version control is a critical tool to facilitate software development workflow and lifecycle, especially when there are multiple contributors.

> In software development, distributed version control (also known as distributed revision control) is a form of version control in which the complete codebase, including its full history, is mirrored on every developer's computer.[1] Compared to centralized version control, this enables automatic management branching and merging, speeds up most operations (except pushing and fetching), improves the ability to work offline, and does not rely on a single location for backups.[1][2][3] Git, the world's most popular version control system,[4] is a distributed version control system.

> In 2010, software development author Joel Spolsky described distributed version control systems as "possibly the biggest advance in software development technology in the [past] ten years".

This section of the Pro Git book gives a nice summary of what version control is all about.

>What is “version control”, and why should you care? Version control is a system that records changes to a file or set of files over time so that you can recall specific versions later. For the examples in this book, you will use software source code as the files being version controlled, though in reality you can do this with nearly any type of file on a computer.

>If you are a graphic or web designer and want to keep every version of an image or layout (which you would most certainly want to), a Version Control System (VCS) is a very wise thing to use. It allows you to revert selected files back to a previous state, revert the entire project back to a previous state, compare changes over time, see who last modified something that might be causing a problem, who introduced an issue and when, and more. Using a VCS also generally means that if you screw things up or lose files, you can easily recover. In addition, you get all this for very little overhead.

So as we see, version control is useful for many types of software development including static web sites,  complex dynamic web applications, commercially available binary packages, and the Linux kernel. In fact, it doesn't have to be software at all and could be any set of binary and/or text files that need to be version controlled.

### Git Fundamentals

It's great that thousands of professional software devlopers use Git to build and maintain the Linux kernel's 40MM+ lines of code, but let's discuss this in practical terms for simple projects and novices like myself. My first example will entail using the open-source Git command-line tool to create and interact with a Git Repository which stores all the files for this blog. Here come the fundamentals.

#### Git Setup

I'll assume you can get Git installed from [these instructions](https://git-scm.com/downloads). At that point, you'll have a set of CLI commands available to interact with Git. You will then want to perform some initial configuration, which typically sets what you want to display as your username for activity attributed to you in Git.

```
git config --global user.name "your-username"
git config --global user.email "your-email"
```

#### Repositories

Git repositories are the containers for individual projects that are version controlled by Git. We'll start with a local Git repository, which is simply a folder on your machine that has been initialized with the `git init` command run within it. To create a local Git repo, either create a new folder or change to an existing one where you want the repository to live.

![](/images/git-init.png)

![](/images/git-tree.png)

Easy enough, we can see that it created a hidden .git directory with a handful of files and sub-directories. This is how an empty Git repo initially begins, and there is currently nothing that for it to keep track of.

#### Git Commit

Commits are discrete point in time snapshots of a repository as it changes. To create commits, we first create or move files into the repo and tell Git to start keeping track of them. You can see I used `touch` to create `example.txt` and `example2.txt`, and then ran `git status`. Git replies that it sees those files present in the repo directory, but it is not tracking them yet. After `git add .`, those files are now being tracked and are in a Staging state, so that they are marked to be included during the next Commit.

![](/images/git-commit.png)

Next I use `git commit -m "Commit comment here` to actually affect the commit, and this snapshot of changes has been recorded in Git. If you issue only `git commit` without the comment argument, Git will open your editor and ask you to input the commit comment right there. Then I made a change to example2.txt, ran `git status` to see that Git noticed it has changed, staged it with `git add`, and made my second commit to this repo. It might not initially make sense that one must add files to Git repeatedly, so think of `git add` as adding a certain file in a certain state, and know that any time you want to record the changed state, you must `git add` the file again.

![](/images/git-commit.png)

![](/images/git-log.png)

`git log` now shows me the history of commits I've made, and you can now see why we set username and email address in Git's config, so that it can be recorded in the commit history. Notice the yellow string after the word commit. This indicates a unique identifier for each exact point in time snapshot of the repo corresponding to each commit and these strings end up being used to navigate between commits as you work within a repo.

#### HEAD Pointer

The concept of the [HEAD pointer](https://blog.git-init.com/what-is-head-in-git/) is important to understanding the state of repos. HEAD moves around depending on what commit (snapshot) of the code base we've asked Git to work with. In the screenshots above, notice **HEAD --> master**. **master** is the name of the default **branch** of our repo. We haven't discussed branches yet, so right now this just tells us our local repo is working with the most recent commit of the teo possible commits we made, in the only branch we have.

>HEAD answers the question: Where am I right now in the repository? It is a pointer to the currently checked-out branch or commit, which contains an immutable snapshot of your entire code base at a given time. Whichever commit HEAD is referencing directly (using the hash) or by reference (using a branch), it'll always be the commit on which any local changes are based.

#### Simple Workflow

{{< mermaid >}}
graph LR;
A[git init]-->B[File changes]-->C[git add .]-->D[git commit -m]-->B;
{{< /mermaid >}}

#### Git Diff

We can use commit strings to look at the state of the repo between two commits. Below `git log` shows I've now made 3 commits, and I run some `git diff`'s between them. The screenshot first shows all the changes from my first commit when the text files were empty, and then the changes between the first and second commits.

![](/images/git-log2.png)

![](/images/git-diff2.png)

#### Git Checkout

Now assume we want to 

#### Git Revert

#### Git Branch

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