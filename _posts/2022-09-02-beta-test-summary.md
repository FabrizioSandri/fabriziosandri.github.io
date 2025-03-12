---
toc: false
layout: post
description: The results of beta testing RcppDeepState-action 
badges: true
tags: [Google Summer of Code, GitHub Action]
comments: false
title: Beta testing results
related_posts: false
---

## Introduction
Beta testing is the process of evaluating a software prototype right before its
release in order to detect flaws. This procedure can be appropriately conducted 
by a third party who is using the program for the first time, the beta tester. 
It's important to note that the most important part of this phase is observing 
the user's interactions with the program without any input from the developers 
in order to identify the most critical sections that need to be handled. 

Thanks to my mentor, I was able to take part in several live beta testing 
sessions where I was the observer and my mentor was the beta tester. The main
goal of this session was to uncover weaknesses and difficulties in the technique
for configuring RcppDeepState-action. As an observer, I couldn't participate;
all I could do was take notes on potential difficulties. This session showed me 
that no matter how much time I spent looking for bugs in the code I wrote, I
would never find them in a short period of time; beta testing, on the other 
hand, allowed me to rapidly detect flaws. 

## A new strategy
Live sessions are extremely beneficial, but they are not the only strategy to 
beta test the code. Another possibility for speeding up the process on a large 
scale is to use automated techniques to test the code and compare the results to
others that we already have access to. This is exactly what I talked about in my
earlier blog post about beta testing RcppDeepState-action on GitHub-hosted CRAN 
packages[^1].

To recap the previous blog posts, I wrote two script:
* one that automatically initializes the RcppDeepState-action on a list of 
preset GitHub repositories that included problems according to Akhila's 
article[^2]. This test were ran within the forked version of the repository to 
ensure that there were no permission issues with the GitHub tokens[^1];
* the other one that automatically replicates the RcppDeepState reports as a 
comment on the remote repositories[^3]. 

Using this method, I was able to identify a number of new issues with the 
RcppDeepState library. The most problematic one I discovered was that 
RcppDeepState would not build the shared object file in some circumstances due 
to missing dependencies in the tested packages. The incorrect usage of 
`R CMD INSTALL` inside the package to build the shared object file without first
installing the missing package dependencies caused this issue. This problem was 
also reported to me by Dr.Martin R. Smith, author of numerous R packages 
accessible on CRAN, like [Quartet](https://cran.r-project.org/web/packages/Quartet/index.html) 
and [TreeSearch](https://cran.r-project.org/web/packages/TreeSearch/).

Furthermore, while examining the TreeSearch package, Dr. Smith identified 
another issue:  one of its packages was not analyzed because some datatypes fell
outside of the list of datatypes supported by RcppDeepState. He asked if this 
was expected behavior, and I said yes, as well as providing him with a detailed 
description of the error's motivation, as well as a link to the RcppDeepState 
wiki article on how to add support for a new datatype. Dr. Smith then made a 
pull request[^4] to the RcppDeepState repository, asking for the `IntegerMatrix`
datatype to be added to RcppDeepState. Because of this, I was able to determine 
whether or not the method required to add support for a new datatype is properly
documented; based on the results, I can claim that Dr.Smith did an excellent job 
by following all of the wiki steps one by one. 

## Conclusion
Finally, I can say with certainty that beta testing really helped me in 
determining what the issues are with RcpDeepStare. Specifically, because to this
new technique, I was able to collect feedback from beta testers, like in the 
case of Dr.Smith. 

<hr />

[^1]: [Beta test RcppDeepState-action on GitHub hosted CRAN packages](https://fabriziosandri.github.io/gsoc-2022-blog/github%20action/2022/08/23/rcppdeepstate-beta-test.html)
[^2]: [Rcpp-based packages where RcppDeepState found issues](https://akhikolla.github.io./packages-folders/)[^1]. 
[^3]: [Issue with RcppDeepState-action beta test](https://fabriziosandri.github.io/gsoc-2022-blog/github%20action/2022/08/30/rcppdeepstate-beta-test-2.html)
[^4]: [IntegerMatrix support](https://github.com/FabrizioSandri/RcppDeepState/pull/26)