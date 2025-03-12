---
toc: false
layout: post
description: A detailed description of debugging Rcpp functions outside of the R environment using Valgrind
badges: true
tags: [Google Summer of Code, Rcpp, Valgrind]
comments: false
title: Debugging Rcpp outside of R
related_posts: false
---

# Introduction
Sometimes it could be necessary to test your Rcpp defined code using an external tool, like Valgrind or GDB. This process however will almost likely come to a crash of your program with a strange Segmentation Fault error. The error is due to the fact that executing Rcpp code is only possible within a working R environment. I talked about this with a Rcpp community member and his answer is reported in this issue [#1221](https://github.com/RcppCore/Rcpp/issues/1221).

> "All" that `Rcpp` does is to provide `R` with callable code via the `.Call()` interface which is meant to _extend a running R session_. Nowhere in the `R` (or `Rcpp`) documentation is it hinted that you can run code separately. Which is why we run all tests etc from `R`.

So, the question might appear straightforward, but is it feasible to run Rcpp code outside of a R environment?
Using RInside, the answer is yes. 

## Proof of concept
Imagine you have the following `bootstrap` function and want to simply execute it and get the result, that you expect to be `10`.
```c++
#include <iostream>
#include <Rcpp.h>

// [[Rcpp::export]]
int bootstrap() {
    // Allocate a sample NumericVector
    Rcpp::NumericVector sample {10,20,30,40,50};
    
    return sample[0];
}

int main(int argc, char* argv[]){
    std::cout << bootstrap() std::endl;
    return 0;
}
```

If you try to compile and execute this program however the result is quite different:
```bash
$ g++ -lR -I"/usr/include/R/" -I/usr/local/include  -I"/home/fabri/R/x86_64-pc-linux-gnu-library/4.2/Rcpp/include" -L/usr/lib64/R/lib -o bootstrap bootstrap.cpp
$  ./bootstrap
[1]    23848 segmentation fault (core dumped)  ./bootstrap
```

### Solution
The solution is to embed in your code a working R environment within the process. This can be done using the powerful RInside library. All you have to do is to include the header files and create an embedded R instance. The preceding example will be modified as follows: 
```c++
#include <iostream>
#include <Rcpp.h>
#include <RInside.h>

// [[Rcpp::export]]
int bootstrap() {
    RInside R;
    // Allocate a sample NumericVector
    Rcpp::NumericVector sample {10,20,30,40,50};
    
    return sample[0];
}

int main(int argc, char* argv[]){
    std::cout << bootstrap() std::endl;
    return 0;
}
```

There will be no errors if you try to compile and run the program after this update. 
```bash
$ g++ -g -lR -lRInside -I"/usr/include/R/" -I/usr/local/include -I/usr/lib/R/library/RInside/include -I"/home/fabri/R/x86_64-pc-linux-gnu-library/4.2/Rcpp/include" -L/usr/lib/R/library/RInside/lib -Wl,-rpath=/usr/lib/R/library/RInside/lib -L/usr/lib64/R/lib -o bootstrap bootstrap.cpp
$  ./bootstrap
10
``` 


## Debugging with Deepstate and valgrind 
Now that we have all of the necessary tools, we can start developing our first Rcpp function test harness. Imagine that we want to fuzz test a function named `getFirstElement` that returns the first element of an Rcpp IntegerVector. Deepstate might be used to send some randomly initialized vectors to this method in order to discover any memory issues. To do this, we may create a basic test harness that employs a function that produces a random integer vector, in this instance `randomIntegerVector`, and then passes that vector to the `getFirstElement` function. This is the final result.
```c++
#include <iostream>
#include <Rcpp.h>
#include <RInside.h>

#include <deepstate/DeepState.hpp>
using namespace deepstate;

// [[Rcpp::export]]
int getFirstElement(Rcpp::IntegerVector v) {
    
    if (v.size() != 0){
        return v[0];
    }

    return -1;
}

// random IntegerVector generation procedure
Rcpp::IntegerVector randomIntegerVector(int maxSize){

    int size = DeepState_IntInRange(1,maxSize);
    Rcpp::IntegerVector vec(size);

    for (int i=0; i<size; i++){
        vec[i] = DeepState_IntInRange(0,1000);
    }
  
    return vec;
}


TEST(Unit, name){
    static RInside R;
    
    Rcpp::IntegerVector randomVec = randomIntegerVector(20);

    getFirstElement(randomVec);

}
```

It's worth noting that the RInside instance has been defined as static, this will prevent error such as `R is already initialized`. Making the `R` variable static force the application to use the same RInside object for all of the application's lifetime.

Let's now compile the harness and run it with the Valgrind Memcheck tool
```bash
$ g++ -g -ldeepstate -lR -lRInside -I"/usr/include/R/" -I/usr/local/include -I/usr/lib/R/library/RInside/include -I"/home/fabri/R/x86_64-pc-linux-gnu-library/4.2/Rcpp/include" -L/usr/lib/R/library/RInside/lib -Wl,-rpath=/usr/lib/R/library/RInside/lib -L/usr/lib64/R/lib -o getFirstElement getFirstElement.cpp
$ valgrind  ./getFirstElement --fuzz --timeout=10
```

This will be the result
```yaml
==43210== Memcheck, a memory error detector
==43210== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==43210== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==43210== Command: ./getFirstElement --fuzz --timeout=10
==43210== 
INFO: Starting fuzzing
WARNING: No seed provided; using 1654880923
WARNING: No test specified, defaulting to first test defined (Unit_name)
INFO: Done fuzzing! Ran 1828 tests (182 tests/second) with 0 failed/1828 passed/0 abandoned tests
==43210== 
==43210== HEAP SUMMARY:
==43210==     in use at exit: 51,299,862 bytes in 9,936 blocks
==43210==   total heap usage: 30,813 allocs, 20,877 frees, 87,993,209 bytes allocated
==43210== 
==43210== LEAK SUMMARY:
==43210==    definitely lost: 0 bytes in 0 blocks
==43210==    indirectly lost: 0 bytes in 0 blocks
==43210==      possibly lost: 0 bytes in 0 blocks
==43210==    still reachable: 51,299,862 bytes in 9,936 blocks
==43210==                       of which reachable via heuristic:
==43210==                         newarray           : 4,264 bytes in 1 blocks
==43210==         suppressed: 0 bytes in 0 blocks
==43210== Rerun with --leak-check=full to see details of leaked memory
==43210== 
==43210== For lists of detected and suppressed errors, rerun with: -s
==43210== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

# Conclusion
We were able to run a debugging tool, specifically the Valgrind suite's Memcheck tool. This demonstrates how integrating many techniques may be an effective method of solving a problem. 