---
toc: false
layout: post
description: An introduction to GitHub Actions and some consideration before writing the GitHub Action for RcppDeepState
badges: true
tags: [Google Summer of Code, GitHub Action]
comments: false
title: RcppDeepState's GitHub Action considerations 
---

## Introduction
Continuous integration and Continuous delivery, also called CI/CD, are two useful methodologies that every developer have hear about. 
This terms refers to the possibility of continuously perform some task with your code in an automated way. 

Continuous integration, often, relies on executing automatic testing to check that an application is correctly behaving. On the other hand, continuous delivery refers to the process of automatically move the code from a testing environment to a production environment. 

These two methodologies, can be implemented in any Git repository using tools such as GitHub Actions and TravisCI. I will focus only on GitHub Actions, which is a feature implemented by the GitHub platform, which allows to create complex workflows in any kind of repository.

#### GitHub Actions 
GitHub actions are fully integrated with GitHub and can respond to any specified GitHub event (new issue, pull request, push on a branch). This is a fully automated engine and GitHub Actions comes with a lot of pre-defined workflows that are provided by communities. A powerful functionality provided by GitHub actions is the possibility to run the code on different operating systems, such as Linux, Windows, macOS and Docker containers.

GitHub Actions are composed of three components: an event, a workflow and an action. Everything start with an event that triggers the execution of a workflow, which is composed of different jobs to compute some operations. Actions instead are reusable units of code that are called within a workflow with the `with` parameter.


## Github Actions and RcppDeepState
In the previous posts, I demonstrated the efficacy of RcppDeepState in detecting issues in Rcpp-based libraries. Based on this, in order to make it easier for developers to execute RcppDeepState on any R package hosted in a GitHub repository, it will be useful to integrate RcppDeepState inside a GitHub Action.
This is exactly the main goal of my Google Summer Of Code project, but before getting started on the development of the GitHub Action, there are a few things to take into account. 

#### Considerations 
Here is a list of considerations that will help me during the development of the RcppDeepState GitHub Action.
1. The first thing I need to take into account is the choice of the architecture for the GitHub Action. According to the GitHub documentation[^1] there are three ways to create an Action:
    * Docker container Action 
    * Javascript Action
    * Composite Action  
2. While developing the Action I need to consider that all the optimization parameters must be disabled in order to get the best performance of RcppDeepState. The motivation behind this choice is explained in the [Issue #8](https://github.com/FabrizioSandri/RcppDeepState/issues/8).
3. Define the inputs and the outputs of the Action.
4. The Action will likely be published on the GitHub Marketplace in the future so that developers can easily find it on the GitHub platform. As a result, I decided it would be better to build it in a different repository. This will, as stated in the documentation[^2], decouple the versioning of the action from the versioning of the RcppDeepState application code. 
> If you're developing an action for other people to use, we recommend keeping the action in its own repository instead of bundling it with other application code. This allows you to version, track, and release the action just like any other software.
> 
> Storing an action in its own repository makes it easier for the GitHub community to discover the action, narrows the scope of the code base for developers fixing issues and extending the action, and decouples the action's versioning from the versioning of other application code.

#### Development
As mentioned in the Considerations section, this GitHub Action will be entirely developed in a separate repository that can be found at the following link: [RcppDeepState-action](https://github.com/FabrizioSandri/RcppDeepState-action).


[^1]: [GitHub documentation: Creating actions](https://docs.github.com/en/actions/creating-actions)
[^2]: [GitHub documentation: Choosing a location for your action](https://docs.github.com/en/actions/creating-actions/about-custom-actions#choosing-a-location-for-your-action)