---
toc: false
layout: post
description: Considerations review
badges: true
tags: [GitHub Action]
comments: false
title: Moving to a composite GitHub Action
---

## Introduction
In the fifth week of work I've created a first working prototype of the GitHub Action for RcppDeepState, based on the considerations that I published in the dedicated blog post[^1]. Within this post, I discussed three different architectures that might be used when creating a GitHub Action. Among the three possibilities, I selected to develop a GitHub action based on Docker.

However, in the pull request for the Action's initial prototype, I indicated the potential of switching from a Docker Action to a Composite Action[^2]: 
> "In the future, I'll also think about if I can express this Action as a composite action."

## Docker system motivation
The key reason for using a Docker-based system is that it enables for the creation of a fully customized environment in which RcppDeepState will execute. This allows me to install only the essential tools and configure the environment such that RcppDeepState fits in. In particular the idea came by the fact that RcppDeepState requires that optimization options must be disabled in the system in order to work with it's maximum performance. Furthermore, because RcppDeepState can only be executed on Linux-based computers, limiting it to a Docker system based on a Linux container is a reasonable option. 

## Problem
However, not everything works well with the Docker system since it lacks the final user interaction. This is a crucial consideration because it is what the final user will see. If the user is unable to detect or rapidly identify the errors in a package, the entire Action will be useless. Using the docker architecture, the action can only print the results in the log in text format, without allowing the results to be uploaded as an artifact file and shared with other steps/jobs in the workflow. 

## Solution
Starting with this I searched for a solution to the problem and discovered that `actions/upload-artifact` allows to share the analysis findings with other jobs by publishing an artifact file. The `actions/upload-artifact` would be handy for RcppDeepState since it may be configured to upload the analyzed package's 'inst/testfiles' folder, which contains the inputs as well as the Valgrind outputs of the RcppDeepState execution. 

The issue is that publishing an artifact file via the Docker system is a challenging operation. One possibility is to combine the present Docker system with a Composite Action to gain the benefits of both systems. Thus in this week of work(seventh week) I will integrate the existing docker system with a composite action. The primary action will be the composite one, and the docker action will be triggered by a composite step. 

<hr />

[^1]: [RcppDeepState's GitHub Action considerations](https://fabriziosandri.github.io/gsoc-2022-blog/github%20action/2022/06/28/github-actions.html)

[^2]: [First prototype](https://github.com/FabrizioSandri/RcppDeepState-action/pull/1)