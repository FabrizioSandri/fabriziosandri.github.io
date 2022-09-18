---
toc: false
layout: post
description: Guide to provide RcppDeepState-action a custom test harness for a function that cannot be analyzed.
badges: true
tags: [RcppDeepState, GitHub Action]
comments: false
title: Provide a custom test harness to RcppDeepState GitHub Action
---


## Introduction
Sometimes it can happen that a function in your package cannot be analyzed
because of some datatypes falling outside of the set of supported one. 
In this scenario, RcppDeepState cannot create the test harness automatically, 
thus the user must supply it manually.

This blog post is meant to be a guide for the users who wants to provide a 
custom test harness to the RcppDeepState GitHub Action.

## Custom test harness
Creating a custom test harness is a difficult operation that must be completed appropriately by adhering to the general test harness structure outlined in the dedicated blog post[^1]. If the test harness is not properly designed, RcppDeepState will not be able to locate and perform the tests. 

Before continuing to read this post, you should first read about the test harness structure that RcppDeepState accepts. The following snippet of code contains the harness file's structure that the user must follow to correctly operate with RcppDeepState. All of this has already been mentioned in the the dedicated blog post[^1].

```c++
#include <fstream>
#include <RInside.h>
#include <iostream>
#include <RcppDeepState.h>
#include <qs.h>
#include <DeepState.hpp>

RInside Rinstance;

/** FUNCTION SIGNATURE
* signature of the function being analyzed must be added here 
*/

/** INPUTS
* here you can define all the inputs for your function, and initialize them
* with a random value generator. Define your inputs inside the `#define INPUTS` 
* macro.
* 
* Example: initialize a random integer parameter 'arg1':
* #define INPUTS \
*   int arg1 = DeepState_Int();
*/

TEST(<package_name>, generator){
  INPUTS
}

TEST(<package_name>, runner){
  INPUTS

  /** INPUTS DUMP
  * for each input defined above you have to save it in the 'inputs' directory
  * created before using the function qs::c_qsave(param, "./inputs/arg_name.qs", 
  * "high", "zstd", 1, 15, true, 1). Remember to replace 'arg_name' inside 
  * "./inputs/arg_name.qs" with the name of the associated input argument.
  * 
  * Example: save the 'arg1' input defined before:
  * qs::c_qsave(arg1, "./inputs/arg1.qs", "high", "zstd", 1, 15, true, 1)
  */

  try{
    /** FUNCTION INVOCATION
    * here you have to add the invocation of the function being analyzed with 
    * all its parameters defined in the section PARAMETERS.
    */ 
  }catch(Rcpp::exception& e){
    std::cout<<"Exception Handled"<<std::endl;
  }
}
```

## Provide the test harness to the Action
Once you have created a custom test harness for your function you can provide it to the RcppDeepState GitHub Action by simply creating a subdirectory inside your package's `/inst/testfiles`, named as the function that is being analyzed. 
This folder will  contain the harness file that should be named as the function being analyzed plus the `_DeepState_TestHarness.cpp` suffix. RcppDeepState will look for existing test harness files and use them instead of generating new ones before performing the analysis step. 

While doing so, keep in mind the naming standards allowed by RcppDeepState; otherwise, unexpected problems may occur. 

#### Example
Assume that a package stored in the root of a repository has a function named `unsupported_datatype` that needs to be analyzed with RcppDeepState. Suppose this function has a parameter whose type falls out of the supported datatypes: the `Rcpp::LogicalVector` datatype.
```c++
#include <Rcpp.h>
using namespace std;

// [[Rcpp::export]]
int unsupported_datatype(Rcpp::LogicalVector param){
  return param.size();
} 
```

Because the test harness cannot be built automatically, this function cannot be analyzed using RcppDeepState. As mentioned above, one solution to this problem is to manually generate the test harness. 

To accomplish this, the user can create a custom test harness and save it in the
`/inst/testfiles/unsupported_datatype/unsupported_datatype_DeepState_TestHarness.cpp` 
location (relative to the repository's root). 

## Conclusion
We saw in this post how to pass a manually created test harness to RcppDeepState-action.
This RcppDeepState feature allows developers to analyze functions with unsupported datatypes as arguments. 

<hr />

[^1]: [Write a custom test harness for functions with unsupported datatypes](https://fabriziosandri.github.io/gsoc-2022-blog/rcppdeepstate/2022/07/29/custom-test-harness.html)