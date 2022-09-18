---
toc: false
layout: post
description: An introduction to Rcpp fuzz testing using RcppDeepState auxiliary functions
badges: true
tags: [RcppDeepState]
comments: false
title: Introduction to RcppDeepState
---

# Introduction to RcppDeepState
In previous posts, we've shown how to use DeepState in conjunction with Valgrind to create a tool that can detect more subtle programming errors in the code. In this post, we'll introduce RcppDeepState, which will add another brick to this toolkit.

## Rcpp 
The Rcpp package[^1] gives R developers a way to write C++ code for some of its modules. It's sometimes useful to write C++ code in place of R functions; it's a performance trade-off. It has been demonstrated that R is particularly inefficient in computing recursive functions when compared to C++. 

It's not the purpose of this article to discuss the Rcpp library, however what you need to know in order to continue is that Rcpp allows to call some external C++ functions exported to the R environment by tagging them with the `// [[Rcpp::export]]` comment on top of them.

### Installation
The installation of the library is quite a straightforward task since the package is available on CRAN.
```R
install.packages("Rcpp")
```

### Sample usage
Let's look at a simple example of a Fibonacci recursive function written in C++ and exported to the R environment to have a better idea of how Rcpp works.
Let's start with a C++ definition of the Fibonacci function. 

```c++
int fibonacci(int n) {
    if (n <= 1)
        return 1;
    return fib(n-1) + fib(n-2);
}
```

The first step to integrate this function with the R environment is to include the appropriate header of Rcpp with it's associated namespace. The final step is to export the `fibonacci` function. As mentioned earlier to do this we use the `// [[Rcpp::export]]` comment. By doing this way, the R environment can easily understand which function should be exported. The final result should look somewhat like this:
```c++
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
int fibonacci(int n) {
    if (n <= 1)
        return 1;
    return fibonacci(n-1) + fibonacci(n-2);
}
```
We don't need to compile anything in order for this function to work; all we have to do in our R environment is include the appropriate library and call the `sourcCpp` function. This function automatically compiles the C++ code and sources it into the running environment, in the same way the standard `source` function works.
```R
> require("Rcpp")
> sourceCpp("./fibonacci.cpp")
> fibonacci(5)
[1] 8
``` 

### Rcpp based package
In the previous example we ran a single C++ file, however most of the times Rcpp is bundled inside packages. This allows users to use this packages without actually having any C++ development tool. To initialize a package using Rcpp, we use the `Rcpp.package.skeleton` function.
If you want to generate a package based on the previous Rcpp fibonacci script, simply supply a list of C++ files to the skeleton function's `cpp_files` parameter.  
```R
Rcpp.package.skeleton("Fibonacci", example_code = FALSE, cpp_files = c("fibonacci.cpp"))
```
This way we end up with a folder containing a working Rcpp based module:
* Rcpp source file are located in `src`
* R files are located in `R`
* Rd documentation files are located in `man`

## RcppDeepState
Let's get started with the main topic of this article: RcppDeepState. RcppDeepState is a combination of three different tools: Rcpp, DeepState, and Valgrind. RcppDeepState's purpose is to perform fuzz testing on Rcpp-based modules. The related work is restricted to R-level fuzzing and does not include fuzz testing of compiled C++ code developed with Rcpp. The lack of such support for fuzz testing led Akhila Chowdary [^2] to create such a powerful tool. 

### Installation process
Since RcppDeepState is not available on CRAN, it must be installed manually.
We can use the traditional devtools way to download the package from the official repository and install it:
```R
install.packages("devtools")
require("devtools")
devtools::install_github("akhikolla/RcppDeepState")
```

In alternative it's possible to directly download the source and install the package from a working R environment:
```R
install.packages("/path/to/RcppDeepState", repos = NULL, type="source")
```

### Steps
In order to perform fuzz testing, RcppDeepState follows the same steps we have used in the previous posts, with a few more steps in between:
* find the exported Rcpp functions
* DeepState test harness creation
* package analysis using Valgrind 

Some steps are left out, such as the automatic test harness creation. We will see this later on in another post.

### RcppDeepState auxiliary functions
The objective of RcppDeepState is to create a link between Rcpp and DeepState in order to perform fuzz testing. This can be done by using some C++ auxiliary function that initializes all the necessary symbolic variables. Symbolic variables were already discussed in a previous article [^3]. 

A list of common auxiliary functions is reported in the following list:
* `RcppDeepState_int()`: generates an integer symbolic variable. Optionally, the parameters `int low, int high` can be used to restrict the generated numbers in a range
* `RcppDeepState_double()`: generates a double symbolic variable. Optionally, the parameters `double low, double high` can be used to restrict the generated numbers in a range
* `RcppDeepState_IntegerVector`: generates a vector of integers with a size in the range 0-100 (inclusive). Optionally, the parameters `int size, int low, int high` can be used to specify the vector size and to restrict the generated numbers in a range.
* `RcppDeepState_NumericVector()`: generates a vector of doubles with a size in the range 0-100 (inclusive). Optionally, the parameters `int size, int low, int high` can be used to specify the vector size and to restrict the generated numbers in a range.
* `RcppDeepState_NumericMatrix()`: generates a matrix of doubles with a size in the range (rows=0,cols=0) and (10,10). Optionally, the parameters `int row,int column,int low,int high` can be used to specify the matrix size (in terms of rows and cols) and to restrict the generated numbers in a range.
* `RcppDeepState_CharacterVector()`: generates a vector of characters with a size in the range 0-100 (inclusive). 
* `RcppDeepState_string()`: generates a string of a length up to 26 characters taken from the alphabet.

This functions can be used in the test harness `TEST` procedure. Let's use an example to further understand this. 

### RcppDeepState TestHarness sample
RcppDeepState has an automated test harness generation function, `deepstate_harness_create()`. However, we manually create a TestHarness using the auxiliary functions indicated above in order to understand how RcppDeepState works behind the scenes. 

Assume you have a leaked function that takes an integer as an input and allocates some Heap memory to it. Consider the case where the developer failed to free the used memory if the integer parameter is more than `500`. Imagine that for some strange motivation the developer forgot to free the used memory if the integer parameter is greater than `500` for some strange reason. The following is an example of the function:
```c++
void copyAndFreeVector(int source){
    // allocates some space for 1 integer
    int* sample = (int*) malloc(sizeof(int));
    if (source < 500){
        free(sample);
    }
}
```

A TestHarness can be generated for this program, using the auxiliary functions above. It can be simply done by creating a symbolic integer variable, initialized using the appropriate `RcppDeepState_int(int low, int high)` function. As you can see, in order to work with Rcpp and RcppDeepState you need to include the necessary libraries. 
```c++
#include <deepstate/DeepState.hpp>
#include <Rcpp.h>
#include <RcppDeepState.h>

using namespace deepstate;

// [[Rcpp::export]]
void copyAndFreeVector(int source){
    // allocates some space for 1 integer
    int* sample = (int*) malloc(sizeof(int));
    if (source < 500){
        free(sample);
    }
}

TEST(IntAllocation, AllocateIntegerWithoutFree) {
    int v = RcppDeepState_int(0, 1000);
    copyAndFreeVector(v);
}
```

Now that the test harness is completed we can compile it, and run the fuzz testing with Valgrind. The compilation needs a lot of library and headers inclusion in order to properly work. We need to include the headers files for R, Rcpp, RcppDeepState and RcppArmadillo. In addition we need to specify to `g++` where to find the shared objects (.so files) for R and RInside. The `-g` option is included to add more debugging information to the compiled binary, which is worth mentioning. 
```bash
g++ -I/usr/include/R 
    -I/usr/lib/R/library/Rcpp/include 
    -I/usr/lib/R/library/RcppDeepState/include 
    -I/usr/lib/R/library/RcppArmadillo/include  
    -L/usr/lib64/R/lib 
    -L/usr/lib/R/library/RInside/lib 
    -lR -lRInside -ldeepstate -o program -g program.cpp
```

The last step is to run the compiled program using Valgrind with the `--leak-check=full` option in order to find memory leaks.
```bash
valgrind --leak-check=full ./program --fuzz --timeout=1
```
This is the result
```yaml
==29579== Memcheck, a memory error detector
==29579== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==29579== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==29579== Command: ./program --fuzz --timeout=1
==29579== 
INFO: Starting fuzzing
WARNING: No seed provided; using 1653991738
WARNING: No test specified, defaulting to first test defined (IntAllocation_AllocateIntegerWithoutFree)
INFO: Done fuzzing! Ran 11711 tests (11711 tests/second) with 0 failed/11711 passed/0 abandoned tests
==29579== 
==29579== HEAP SUMMARY:
==29579==     in use at exit: 23,544 bytes in 5,885 blocks
==29579==   total heap usage: 11,735 allocs, 5,850 frees, 165,956 bytes allocated
==29579== 
==29579== 23,536 bytes in 5,884 blocks are definitely lost in loss record 2 of 2
==29579==    at 0x4845888: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==29579==    by 0x11B414: copyAndFreeVector(int) (program.cpp:10)
==29579==    by 0x11B484: DeepState_Test_IntAllocation_AllocateIntegerWithoutFree() (program.cpp:18)
==29579==    by 0x11B439: DeepState_Run_IntAllocation_AllocateIntegerWithoutFree() (program.cpp:16)
==29579==    by 0x113130: DeepState_RunTestNoFork (in /home/fabri/test/testHarness/RcppTestHarness/program)
==29579==    by 0x113654: DeepState_FuzzOneTestCase (in /home/fabri/test/testHarness/RcppTestHarness/program)
==29579==    by 0x1138E7: DeepState_Fuzz (in /home/fabri/test/testHarness/RcppTestHarness/program)
==29579==    by 0x10EDAA: main (in /home/fabri/test/testHarness/RcppTestHarness/program)
==29579== 
==29579== LEAK SUMMARY:
==29579==    definitely lost: 23,536 bytes in 5,884 blocks
==29579==    indirectly lost: 0 bytes in 0 blocks
==29579==      possibly lost: 0 bytes in 0 blocks
==29579==    still reachable: 8 bytes in 1 blocks
==29579==         suppressed: 0 bytes in 0 blocks
==29579== Reachable blocks (those to which a pointer was found) are not shown.
==29579== To see them, rerun with: --leak-check=full --show-leak-kinds=all
==29579== 
==29579== For lists of detected and suppressed errors, rerun with: -s
==29579== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

As you can see, the program allocates 11,735 integers, but frees only 5,850 of them. Valgrind tells us that the error is due to the `malloc` call in the program at line 10:
```yaml
==29579==    at 0x4845888: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==29579==    by 0x11B414: copyAndFreeVector(int) (program.cpp:10)
```

Let's solve the problem and run fuzz testing again.
```c++
#include <deepstate/DeepState.hpp>
#include <Rcpp.h>
#include "RcppDeepState.h"

using namespace deepstate;

// [[Rcpp::export]]
void copyAndFreeVector(int source){
    // allocates some space for 1 integer
    int* sample = (int*) malloc(sizeof(int));
    free(sample);
}

TEST(IntAllocation, AllocateIntegerWithoutFree) {
    int v = RcppDeepState_int(0, 1000);
    copyAndFreeVector(v);
}
```

With the exception of the 8 bytes classified as `stil reachable`, we don't have any memory leaks this time. I discovered that these 8 bytes are related to the inclusion of the R library (`-lR` argument). 
```yaml
==29423== Memcheck, a memory error detector
==29423== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==29423== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==29423== Command: ./program --fuzz --timeout=1
==29423== 
INFO: Starting fuzzing
WARNING: No seed provided; using 1653991674
WARNING: No test specified, defaulting to first test defined (IntAllocation_AllocateIntegerWithoutFree)
INFO: Done fuzzing! Ran 6335 tests (6335 tests/second) with 0 failed/6335 passed/0 abandoned tests
==29423== 
==29423== HEAP SUMMARY:
==29423==     in use at exit: 8 bytes in 1 blocks
==29423==   total heap usage: 6,359 allocs, 6,358 frees, 144,452 bytes allocated
==29423== 
==29423== LEAK SUMMARY:
==29423==    definitely lost: 0 bytes in 0 blocks
==29423==    indirectly lost: 0 bytes in 0 blocks
==29423==      possibly lost: 0 bytes in 0 blocks
==29423==    still reachable: 8 bytes in 1 blocks
==29423==         suppressed: 0 bytes in 0 blocks
==29423== Reachable blocks (those to which a pointer was found) are not shown.
==29423== To see them, rerun with: --leak-check=full --show-leak-kinds=all
==29423== 
==29423== For lists of detected and suppressed errors, rerun with: -s
==29423== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

## Conclusion
In this post we have seen how to use RcppDeepState auxiliary functions to manually generate a Test Harness. This is a procedure that is always done in an automated way by using `deepstate_harness_create()`. The purpose of this post is to understand how the Test Harness creation procedure works and combine RcppDeepState with a simple Rcpp program.



[^1]: [Rcpp package on CRAN](https://cran.r-project.org/web/packages/Rcpp/index.html)
[^2]: [Akhila Chowdary blog](https://akhikolla.github.io/)
[^3]: [DeepState symbols definition](https://fabriziosandri.github.io/gsoc-2022-blog/deepstate/fuzz/c++/2022/05/25/about-deepstate.html#symbols-definition)