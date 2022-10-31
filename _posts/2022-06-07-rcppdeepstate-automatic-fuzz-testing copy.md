---
toc: false
layout: post
description: An quick overview of package fuzz testing using RcppDeepState 
badges: true
tags: [Google Summer of Code, RcppDeepState]
comments: false
title: Rcpp package fuzz testing with RcppDeepState
---

# Automated fuzz testing
We saw how to manually create a Test Harness for a Rcpp written function in order to run fuzz testing on it in the previous blog article.
In practice, this strategy is never used: the majority of the time, the Harness construction process is automated.
This is what we'll talk about in this blog article. 

## The steps
The RcppDeepState library was introduced in the previous post to do package fuzz testing. This library includes a collection of features that allow you to construct the test harness for a certain package function automatically. 

RccpDeepState includes the following two key functions: 
* `deepstate_harness_compile_run`
* `deepstate_harness_analyze_pkg`

Let's take a closer look at them. 

### The compilation procedure
The function `deepstate_harness_compile_run(package_path)` is used to run the compilation process, and it accepts the location of the library we want to fuzz test.


This step begins by looking for any Rcpp written functions in the supplied library. This is accomplished by looking for information about the package's exported functions, which are found in the `RcppExports.cpp` file in the `src` source folder. Once you've gathered all of the data, you'll need to construct a test harness file. This phase is done by `deepstate_pkg_create`, which generates a Harness file for each function discovered in `<package_dir>/inst/testfiles/<function_name>`. This file includes the headers as well as the 'TEST' function definition, which includes all of the symbolic variables. 


Using the `R CMD INSTALL` command, a shared object for the library is also produced (`.so` file). It's worth noting that the shared object must be compiled with the `-g` option in order to include debug symbols.
I discovered that the shared object was produced without debugging symbols on some Linux systems. I wrote an additional piece of code to rewrite the `Makevars` file holding the compilation parameters in order to solve the problem. This issue is solved in the [Pull request #3](https://github.com/FabrizioSandri/RcppDeepState/pull/3).

The fuzz testing approach begins once the library and harness have been generated: the output of the tests is stored in order to evaluate them later using Valgrind. 

### The analysis phase
Following the completion of the fuzz testing, the next step is to analyze the inputs generated. This is done using `deepstate_harness_analyze_pkg(package_path)`. This function looks for binary files containing the inputs in the `<package_dir>/inst/testfiles` directory. The harness is then rerun with the `--input_test_file` option: this allows to provide an input test case. It's worth noting that in this second stage, the harness is examined by the Valgrind Memcheck tool, resulting in an XML log file. 

RcppDeepState then parses this file for error information (message, line number, etc.) and returns them to the `deepstate_harness_analyze_pkg(package_path)` function.

## Example
RcppDeepState includes a sample library under the `RcppDeepState/inst/testpkgs/testSAN` folder. It can be useful as a toy library for detecting problems. Following the approach outlined in the previous part, we examine the package in this section.


The first step is to use `deepstate_harness_compile_run` to compile the library. This function prints a list of successfully constructed harness when the compilation processes are done. In this example, all of the functions were appropriately built. On the other hand, if any functions are not built, an error message is generated. 
```R
> deepstate_harness_compile_run("RcppDeepState/inst/testpkgs/testSAN")

... compilation steps ...

[1] "rcpp_read_out_of_bound"      "rcpp_use_after_deallocate"  
[3] "rcpp_use_after_free"         "rcpp_use_uninitialized"     
[5] "rcpp_write_index_outofbound" "rcpp_zero_sized_array"  
```

The next step is to use Valgrind's Memcheck tool to perform the analysis. 

```R
> result <- deepstate_harness_analyze_pkg("RcppDeepState/inst/testpkgs/testSAN")
```

We may get a detailed description of the faults detected by printing the contents of `result$logtable` :
```R
> knitr::kable(result$logtable)
|err.kind    |message                |file.line                    |address.msg                                                 |address.trace                |
|:-----------|:----------------------|:----------------------------|:-----------------------------------------------------------|:----------------------------|
|InvalidRead |Invalid read of size 1 |use_after_deallocate.cpp : 8 |Address 0x806cfe5 is 5 bytes after a block of size 0 free'd |use_after_deallocate.cpp : 6 |
|InvalidRead |Invalid read of size 1 |use_after_deallocate.cpp : 7 |Address 0x806cfe5 is 5 bytes after a block of size 0 free'd |use_after_deallocate.cpp : 6 |

|err.kind    |message                |file.line                    |address.msg                                                 |address.trace                |
|:-----------|:----------------------|:----------------------------|:-----------------------------------------------------------|:----------------------------|
|InvalidRead |Invalid read of size 1 |use_after_deallocate.cpp : 8 |Address 0x806cfe5 is 5 bytes after a block of size 0 free'd |use_after_deallocate.cpp : 6 |
|InvalidRead |Invalid read of size 1 |use_after_deallocate.cpp : 7 |Address 0x806cfe5 is 5 bytes after a block of size 0 free'd |use_after_deallocate.cpp : 6 |

|err.kind    |message                |file.line                    |address.msg                                                 |address.trace                |
|:-----------|:----------------------|:----------------------------|:-----------------------------------------------------------|:----------------------------|
|InvalidRead |Invalid read of size 1 |use_after_deallocate.cpp : 8 |Address 0x806cfe5 is 5 bytes after a block of size 0 free'd |use_after_deallocate.cpp : 6 |
|InvalidRead |Invalid read of size 1 |use_after_deallocate.cpp : 7 |Address 0x806cfe5 is 5 bytes after a block of size 0 free'd |use_after_deallocate.cpp : 6 |


```