---
title: "Github Primer (Part 2)"
draft: true
date: "2025-08-19"
series: ["Github"]
series_order: 2
categories: ["Devops", "Cloud"]
tags: ["Github", "CI/CD"]
---

## Branches with Github

Branching with Github in the mix is not a lot different. We still just create local branches as desired like `git checkout -b new-branch`. The extra step is just to `git push -u origin new-branch`.

![](/images/github-add-branch.png)

![](/images/github-new-branch.png)

You can see the additonal branch **new-branch** is now present on Github, and it contains the file I just added to that branch on my local repo before pushing it. Github tells us this new branch is 1 commit ahead of **main**.

When I'm ready to merge **new-branch** into **main**, I just do so locally by `git checkout main` and `git merge new-branch` to merge, then `git push` to push **main** back up to Github. Github notices the 2 branches are back in sync with commits.

![](/images/github-merge.png)

## Github Pull

In addition to pushing changes from local repos to Github, we can pull changes made to Github repos down into our local branches.

![](/images/github-add-file.png)

![](/images/github-pull.png)

## Cloning Github Repos

## Pull Requests


## Working with Forks

## Github Workflow

{{< mermaid >}}
graph TD;
A[Create Branch]-->B[Make Commits]-->C[Open Pull Request]-->D[PR Review & Approve]-->E[Deploy]-->F[Merge]-->A;
{{< /mermaid >}}

Create a Branch: Start new work without affecting the main code.
Make Commits: Save progress as you make changes.
Open a Pull Request: Ask others to review your work.
Review: Discuss and improve the changes together.
Deploy: Test your changes before merging.
Merge: Add your finished work to the main branch.

###