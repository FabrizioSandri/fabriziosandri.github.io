---
toc: false
layout: post
description: The summary of my experience with Google Summer of Code and a brief recap of my work
badges: true
tags: [Google Summer of Code]
comments: false
title: My Google Summer of Code experience 
---

## Introduction
The conclusion of this long and magnificent experience of Google Summer of Code, which began around five months ago, has arrived, and it is now time to recap the points of this long journey. I knew from the first day that this was going to be a fantastic experience, but I never expected to learn so much this summer! 

It all started some months ago when my thesis supervisor advised that I engage in this initiative that Google has been doing for several years. My thesis supervisor put me in touch with a PhD student at the University of Trento who had previously participated in GSOC and is now a mentor. Fascinated by his experience, I set out to work to ensure that my proposal was chosen. After several weeks of effort, I submitted my project idea, and on May 20, I received an email informing me that I had been accepted for Google Summer of Code 2022; I was overjoyed! 

## Google Summer of Code work
The first three weeks of work, known as the Community Bonding period, have been a chance for me to study and become familiar with the tools that I would be using for the next ten weeks. This period was really beneficial to me since I was able to immerse myself into the R community while also learning a lot of new topics thanks to my blog posts: writing them helped me keep track of everything I had been studying. 

In accordance with the [goal of my project](https://github.com/rstats-gsoc/gsoc2022/wiki/RcppDeepState#details-of-your-coding-project) I've been working on two distinct repositories: 
* [FabrizioSandri/RcppDeepState](https://github.com/FabrizioSandri/RcppDeepState) : the RcppDeepState package implementation, which runs fuzz testing with Valgrind and DeepState; 
* [FabrizioSandri/RcppDeepState-action](https://github.com/FabrizioSandri/RcppDeepState-action) : Implementation of the GitHub action that allows developers to execute RcppDeepState on GitHub-hosted Rcpp-based packages. The action is also available on the [GitHub Marketplace](https://github.com/marketplace/actions/rcppdeepstate). 

All of my progress on the two aforementioned repositories is documented in the pull requests and issues listed underneath. Furthermore, the week summaries that I posted at the end of each week give a more detailed explanation of the changes that I made week by week; you can find them [here](https://fabriziosandri.github.io/gsoc-2022-blog/).

##### FabrizioSandri/RcppDeepState-action 
<table class="recap-table">
  <tr>
    <th>Pull requests</th>
    <th>Issues</th>
  </tr>
  <tr>
    <td class="recap-table-cell recap-left-cell">
      <ul>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/1">#1 First prototype</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/2">#2 Exit codes</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/3">#3 Input arguments</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/4">#4 Composite action</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/6">#6 Action&#39;s comments</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/7">#7 Action&#39;s comments - 2</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/8">#8 Additional arguments and documentation</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/9">#9 Docker hub integration</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/10">#10 Report size exceeds the maximum GitHub comment size</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/12">#12 Docker Hub tags problem</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/15">#15 Report details</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/17">#17 Get errors count</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/18">#18 Parameters names/values</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/20">#20 Dependencies issue</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/pull/21">#21 Action logs enhancement</a></li>
      </ul>
    </td>  
    <td class="recap-table-cell">
      <ul>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/issues/2">#2 Segmentation fault not catched</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/issues/4">#4 Fuzzing functions with Rcpp parameters</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/issues/7">#7 Valgrind for initial pass</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/issues/8">#8 RcppDeepState optimization options</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/issues/10">#10 Missing Rcpp Strings support</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/issues/13">#13 Wrong inputs column</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/issues/18">#18 Valgrind and Clang-14 dwarf support</a></li>
      </ul>
    </td>
  </tr>
</table>


##### FabrizioSandri/RcppDeepState 


<table class="recap-table">
  <tr>
    <th>Pull requests</th>
    <th>Issues</th>
  </tr>
  <tr>
    <td class="recap-table-cell recap-left-cell">
      <ul>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/1">#1 Makefile generation fix and improvements</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/3">#3 Override default Makevars</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/5">#5 Harness creation improvements</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/6">#6 Debug symbols tests</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/9">#9 Logging improvements</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/11">#11 Rcpp string support</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/12">#12 Fuzz only supported functions</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/14">#14 qs::c_qsave moved to the runner</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/15">#15 Automatically setup CI</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/16">#16 Harness creation improvements - 2</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/17">#17 Custom test harness</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/19">#19 Harness unit test name</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/20">#20 Editable functions</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/21">#21 Harness unit test name - 2</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/22">#22 Exit status codes</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/23">#23 Harness creation improvements - 3</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/24">#24 Makevars issue</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/25">#25 Dependencies problem</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/26">#26 IntegerMatrix support</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState/pull/27">#27 Dependencies issue</a></li>
      </ul>
    </td>  
    <td class="recap-table-cell">
      <ul>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/issues/5">#5 what to do when auto comments are too large?</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/issues/11">#11 Docker Hub integration trigger event</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/issues/13">#13 SHA in PR comment?</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/issues/14">#14 beta test on other packages?</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/issues/16">#16 parameter names / value?</a></li>
      <li><a href="https://github.com/FabrizioSandri/RcppDeepState-action/issues/19">#19 Not finding source files from DESCRIPTION LinkingTo: field?</a></li>
      </ul>
    </td>
  </tr>
</table>

## Future work
There's still a lot to accomplish with RcppDeepstate, and I'm specifically talking to the fact that a future step would be the package's release on CRAN. As the goal of my project, I spent a lot of time during the Google Summer of Code improving RcppDeepState for a future CRAN publication, but with all of the problems related to Deepstate portability on Windows-based systems, I believe that this is a future step, as demonstrated by the results obtained after uploading the package on Win-builder: most of the errors were caused by the use of a Windows system. 

Another step would be to expand RcppDeepState's datatype coverage by introducing additional datatypes, allowing RcppDeepState to run on a broader range of packages. To do this, I created a wiki article that describes how to [Add a new datatype to RcppDeepState](https://github.com/FabrizioSandri/RcppDeepState/wiki/Add-a-new-datatype-to-RcppDeepState). The scope of this page has been successfully shown thanks to Dr.Martin R. Smith's first external contribution to the RcppDeepState package. More information is available in the [dedicated blog post](https://fabriziosandri.github.io/gsoc-2022-blog/github%20action/2022/09/02/beta-test-summary.html).

Another future improvement would be to provide a new option to RcppDeepState-action that allows users to choose a different fuzzer than the default one provided by Deepstate; Libfuzzer, Manticore and Angr are three alternative fuzzers. 

## My Google Summer of Code experience
My Google Summer of Code experience was incredible, something I never imagined, and it was one of the best in my career. This experience taught me a lot about working with others and contributing to open source. At the end of this experience, I can certainly say that I will spend more time to open source than the time I've spent so far. As a maintainer, I will continue to contribute to my project. 

I am grateful to all of my mentors who took the time to guide me through this journey and share their unique experience and knowledge with me. Special thanks go to: 
* my evaluating mentor Dr.Toby Dylan Hocking, who has always been available to help me during the Google Summer of Code. I was able to stay on track and keep up the good work thanks to his helpful guidance. 
* my mentor Randy Lai, who helped me with his great experience with GitHub Actions to improve the RcppDeepState action;
* my mentor Anirban Chetia, who gave me good suggestions on the challenges I had in the early stages of utilizing RcppDeepState;
* Dr.Martin R. Smith, for his great availability to beta test my GitHub action on his packages and for integrating a new datatype to RcppDeepState; 
* all of the R community people I've interacted with on GitHub, the RcppCore team; 
* Google/Google Summer of Code for organizing this fantastic event year after year. 
