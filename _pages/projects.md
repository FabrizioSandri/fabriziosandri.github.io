---
layout: page
title: Projects
permalink: /projects/
description: List of projects I've been actively engaged in over the years.
nav: true
nav_order: 2
---

#### [RcppDeepState](https://github.com/FabrizioSandri/RcppDeepState)
RcppDeepState is an R library to fuzz test compiled code in Rcpp packages. The
package extends the DeepState framework to fully support Rcpp based packages.
RcppDeepState allows to fuzz test R library's C++ code in order to find
more subtle bugs like memory leaks or even more general memory errors.

- Reduced the time necessary to setup and run RcppDeepstate from more than 30
  minutes to less than 1 minute. This improvement makes it possible for
  RcppDeepState-action to operate quickly inside the runner.
- Enhanced code quality of RcppDeepState and resolved over 20 bugs in the
  library.
- [See my blog](https://fabriziosandri.github.io/gsoc-2022-blog)

#### [RcppDeepState-action](https://github.com/FabrizioSandri/RcppDeepState-action)
I am the author of RcppDeepState-action, a GitHub Action which enables users who
develop R packages using Rcpp to find memory leaks in their code that is hosted
on GitHub.

- As part of the [Google Summer of Code
  2022](https://summerofcode.withgoogle.com/programs/2022/projects/t87xbcg2) I
  developed this GitHub Action for non deterministic fuzz testing with
  RcppDeepState.


#### [R-integration](https://github.com/FabrizioSandri/r-integration)
I am the author of R-integration, a Node.js library that allows to execute
arbitrary R commands or scripts directly from the Node.js environment. This
integration works on Windows and GNU/Linux based systems and uses basic
primitives to access the R binary.


# Open source contributions

#### [DeepState](https://github.com/trailofbits/deepstate)
- I am the author of the entire Windows implementation of the framework. Before
this, DeepState was restricted to POSIX-compliant GNU/Linux or macOS operating
systems. [Merged into DeepState in
2023](https://github.com/trailofbits/deepstate/pull/428)

#### [NASA FPrime](https://github.com/nasa/fprime)
- Resolved potential overflow issue arising from overlapping strings in memory
during copying operations. [Merged into FPrime in
2021](https://github.com/nasa/fprime/pull/1164)
- Improved code quality and conducted reviews of string manipulation framework
  utilities. [Merged into FPrime in
  2021](https://github.com/nasa/fprime/pull/1151)

#### [Italian digital public services](https://github.com/italia/daf-recipes)
- Led team effort to devise a Docker-based system for aggregating and monitoring
  vast data sets.
- Integrated 4 services with Docker-compose reducing to few seconds the time
  needed to aggregate and visualize data. [Merged into daf-recipes in
  2017](https://github.com/italia/daf-recipes/commits?author=FabrizioSandri)


#### R and statistical software community contribution
I incorporated RcppDeepState functionality into packages that can be accessed on
both CRAN and GitHub. This enables the identification and reporting of
memory-related problems within these packages. Through GitHub Actions'
continuous integration system, developers can effortlessly assess their R
packages for memory-related issues, eliminating the need to manually configure
fuzzers and other memory detection utilities.

This is a list of the packages that RcppDeepState-action automatically checks:

- [Quartet](https://github.com/ms609/Quartet) ([CRAN](https://cran.r-project.org/web/packages/Quartet))
- [GUILDS](https://github.com/thijsjanzen/GUILDS) ([CRAN](https://cran.r-project.org/web/packages/GUILDS))
- [PP](https://github.com/jansteinfeld/PP) ([CRAN](https://cran.r-project.org/web/packages/PP/))
- [RcppNumerical](https://github.com/yixuan/RcppNumerical) ([CRAN](https://cran.r-project.org/web/packages/RcppNumerical))
- [Rogue](https://github.com/ms609/Rogue) ([CRAN](https://cran.r-project.org/web/packages/Rogue))
- [TreeDist](https://github.com/ms609/TreeDist) ([CRAN](https://cran.r-project.org/web/packages/TreeDist))
- [TreeSearch](https://github.com/ms609/TreeSearch) ([CRAN](https://cran.r-project.org/web/packages/TreeSearch))
- [TreeTools](https://github.com/ms609/TreeTools) ([CRAN](https://cran.r-project.org/web/packages/TreeTools))


# Coursework

#### [CLIP Region Proposal Network](https://github.com/FabrizioSandri/deep-learning-project)
Visual grounding framework written using PyTorch employing CLIP as the
foundation. This framework was created as a project for the Deep Learning course
at the University of Trento.

- Applied transfer learning using OpenAI CLIP and constructed a Region Proposal
  Network on its foundation. This allowed to leverage the pretrained
  convolutions of CLIP ResNet for extracting region proposals from images.

#### [Hybrid recommendation system](https://github.com/FabrizioSandri/data-mining-project)
Implementation of an advanced recommendation system for the ”Data Mining” course
at University of Trento.

- Developed a hybrid recommendation system combining content-based,
  collaborative filtering, and Locality sensitive hashing to optimize query
  suggestions in a database management system using Python.

#### [Standalone Stellar Blockchain](https://github.com/FabrizioSandri/Standalone-stellar-blockchain)
A ready-to-use setup of a standalone [Stellar blockchain](https://stellar.org/)
based on Docker that I implemented as part of my thesis study.

- Docker-compose implementation of a Stellar validator node that runs on a fully
  private blockchain, separated from the “public” and “test” one.

#### [LR(0) automata generator](https://github.com/FabrizioSandri/LR-0-automa-generator)
Parsing automa generator written in C++ created as final project for the ”Formal
Languages and Compilers” course at University of Trento.

