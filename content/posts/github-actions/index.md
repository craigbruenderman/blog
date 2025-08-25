---
title: "Github Actions"
date: "2025-08-24"
categories: ["Devops", "Cloud"]
tags: ["Github", "CI/CD"]
---

## Overview

Github Actions is Github's CI/CD platform which enables automated build/test/deploy pipelines for Github repos. Workflows are custom defined and are event-triggered by actions taken upon the repo like pull requests.

## Key Components

These are the foundational elements you should understand when starting out with Github Actions.

### Workflows

Workflows are automated processes that run one or more jobs. Workflows are attached to repos, and there can be any number of them in a repo. Workflows are defined in YAML files that lives in your repo in the `.github/workflows` directory. There are reserved keywords that are used to define different parts of the Workflow.

### Jobs

Jobs are contained within workflows, and contain one or more steps that are executed in order. Jobs themselves run in parallel by default, but can be configured to run sequentially as well as conditionally. Again, there can be any number of jobs in a workflow.

### Steps

Steps are the specific individual tasks contained within a job. The Steps themselves are either shell commands, or Github defined Actions. Any number of steps can be defined in a job, and they run in the order they are defined. Steps can be conditional, like Jobs.

### Events

Events are the trigger conditions that cause workflows to execute. Events could be manual actions or things such as a push to the repo.

### Runners

Runners are execution environments that jobs run within. Github offers [pre-built Runners](https://docs.github.com/en/actions/concepts/runners/github-hosted-runners) for MacOS, Linux, and Windows operating systems to execute steps within, and custom Runners can also be created and used.

## Hugo Blog Publishing Use Case

One of the things I use Github actions for is to process and publish this very blog. The process goes like so:

{{< mermaid >}}
graph TB;
A@{ shape: docs, label: "Draft posts in Markdown locally"}-->B@{ shape: trap-b, label: "Review by running hugo server locally"};
B-->C[Commit drafts to local repo];
C-->D[Push local repo to Github];
D-->E@{ shape: processes, label: "Github Action triggered by push Event"};
E-->F@{ shape: lin-cyl, label: "Rendered blog content published via Github Pages"};
{{< /mermaid >}}

### Keywords Used in Workflow file

* The **name** keyword to define the name of the Workflow
* The **on** keyword defines the Events which trigger the Workflow execution
* The **jobs** keyword has job names nested underneath it with the **runs_on** keyword to indicate which Runner execution environment to use
* The **steps** keyword contains key/value pairs of each step
* There is a a **Github Action** used in one of these steps to checkout the blog repo into the Runner
* **deploy** is another Job which uses another Github Action which deploys the resulting Hugo-rendered blog to Github Pages

{{< codeimporter url="https://raw.githubusercontent.com/craigbruenderman/blog/refs/heads/main/.github/workflows/hugo.yaml" type="yaml" >}}

### Setup in Github

The entire setup for using Github Actions to render and publish blogs with Hugo is covered [here](https://gohugo.io/host-and-deploy/host-on-github-pages/). Github provides real-time feedback while Workflows are running.

![](/images/gh-actions-wf-running.png)

In the image, you can see the *build* job completed in 52s, and the *deploy* job has been underway for 2:38s. These job runtimes vary quite a bit, likely due to current load on the environment where the Github Runners execute. I've seen my Workflow finish in under a minute, and over 6 minutes. One of the status indicators you'll see indicates if your jobs are waiting in queue to start.

Once the job has completed, Github gives feedback along with a link to the published blog. I have Github Pages configured with a custom domain, but the blog is indeed hosted via Github Pages.

![](/images/gh-actions-wf-success.png)

## Conclusion

That was a brief look at Github Actions as CI/CD pipeline. There are much moire sophisticated workflows that can be built and my little example did not employ any sort of testing. I'll look to incorporate Github Actions in a future post on a topic that uses Terraform to demonstrate more CI/CD features.