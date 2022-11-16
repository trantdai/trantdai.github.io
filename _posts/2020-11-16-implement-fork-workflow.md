---
title: Implement Fork Worflow
layout: post
post-image: "/assets/images/blog/git-fork-workflow.jpg"
description: Description of a fork, its benefits and git fork workflow implementation
tags:
- howto
- blog
- git
- gitworkflow
- fork
- devops
---

# Overview

A fork is a copy of a repository. Forking a repository allows you to experiment with changes without affecting the original project freely. You can create a fork to suggest changes when you don't have permission to write to the original project directly.

You can fork repositories in the following situations:

- I want to make a change.
- I think the project is exciting and may want to use it in the future.
- I want to use some code in that repository as a starting point for my project.

# The forking workflow

- Create a fork.
- Clone it locally.
- Make your changes locally and push them to a branch.
- Create and complete a PR to upstream.
- Sync your fork to the latest from upstream.

For convenience, after cloning, you'll want to add the upstream repository (where you forked from) as a remote named upstream.
```
git remote add upstream {upstream_url}
```
# Sync your fork to the latest

When you've gotten your PR accepted into upstream, you'll want to make sure your fork reflects the latest state of the repo.

It is recommended rebasing on upstream's main branch (assuming main is the main development branch).

```
git fetch upstream main
git rebase upstream/main
git push origin
```

The forking workflow lets you isolate changes from the main repository until you're ready to integrate them. When you're ready, integrating code is as easy as completing a pull request.

# References
1. [Implement the fork workflow](https://learn.microsoft.com/en-us/training/modules/plan-fostering-inner-source/3-implement-fork-workflow)
2. [Describe inner source with forks](https://learn.microsoft.com/en-us/training/modules/plan-fostering-inner-source/4-describe-forks)
