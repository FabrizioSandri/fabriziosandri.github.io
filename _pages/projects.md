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

#### [RcppDeepState-action](https://github.com/FabrizioSandri/RcppDeepState-action)
I am the author of RcppDeepState-action, a GitHub Action which enables users who
develop R packages using Rcpp to find memory leaks in their code that is hosted
on GitHub.


#### [R-integration](https://github.com/FabrizioSandri/r-integration)
I am the author of R-integration, a Node.js library that allows to execute
arbitrary R commands or scripts directly from the Node.js environment. This
integration works on Windows and GNU/Linux based systems.


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

#### [A Comparative Study of Genetic-Based Approaches for Enhanced Hourly Temperature Predictions](https://github.com/FabrizioSandri/bio-inspired-ai-project)
A comparative study of two genetic-based approaches for enhancing hourly
temperature predictions. The first approach employs a Long Short-Term
Memory(LSTM) network, optimized through Genetic Algorithms for hyperparameter
tuning. The second approach utilizes Genetic Programming to create an
interpretable model. The study showcases the strengths and weaknesses of both
approaches, with the LSTM model outperforming the Genetic Programming model in
terms of Mean Absolute Error, while the Genetic Programming model provides an
interpretable model, which might be preferred in scenarios where computational
cost and efficiency are important. 


#### [CLIP Region Proposal Network](https://github.com/FabrizioSandri/deep-learning-project)
Designed a deep learning framework, ClipRPN, to tackle visual grounding, a
challenge linking language and perception by grounding linguistic symbols in
visuals. Built upon the CLIP model from OpenAI, the solution converts images and
textual descriptions into feature maps and embeddings, which are then fused and
processed by a Region Proposal Network to predict bounding boxes in images based
on the textual information. Implemented and evaluated the model using the
RefCOCOg dataset, showcasing the model's ability to generate accurate bounding
boxes around referred objects in images with an overall intersection over union
(IOU) of 41.19%.


#### [Hybrid recommendation system](https://github.com/FabrizioSandri/data-mining-project)
Designed a cutting-edge query recommendation system focused on revolutionizing
query recommendations within a Database Management System. Leveraged a unique
combination of Collaborative Filtering and Locality Sensitive Hashing (LSH) to
optimize recommendation accuracy and time efficiency. Showcased the seamless
integration of LSH for improved item similarity searches and introduced a
content-based strategy to develop a hybrid recommendation system, resulting in a
improvement in query recommendation accuracy.


#### [Distributed Key-Value Store in Akka](https://github.com/FabrizioSandri/distributed-systems-project)
Inspired by Amazon Dynamo, this project introduces a DHT-based peer-to-peer
key-value storage service implemented using Akka actors. It provides a simple
user interface to upload/request data and issue management commands, emphasizing
key principles of data partitioning, replication, and dynamic network
management.

#### [Standalone Stellar Blockchain](https://github.com/FabrizioSandri/Standalone-stellar-blockchain)
A ready-to-use setup of a standalone [Stellar blockchain](https://stellar.org/)
based on Docker that I implemented as part of my thesis study.

- Docker-compose implementation of a Stellar validator node that runs on a fully
  private blockchain, separated from the “public” and “test” one.

#### [LR(0) automata generator](https://github.com/FabrizioSandri/LR-0-automa-generator)
Parsing automa generator written in C++ created as final project for the ”Formal
Languages and Compilers” course at University of Trento.

