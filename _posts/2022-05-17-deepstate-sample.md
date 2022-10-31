---
toc: false
layout: post
description: A minimal example of using deepstate to perform fuzz testing on a C++ function
badges: true
tags: [Google Summer of Code, DeepState]
comments: false
title: Sample DeepState fuzz test
---

# Sample usage of the Deepstate C++ library
Assume that the NASA FPrime library contains a function that converts a string to a Mixed-Uppercase string. In reality the NASA FPrime contains a lot of functions that deals with strings. 
What is a Mixed-Uppercase string? With this term I am referring to a string which all characters in odd positions must be capitalized. The following is a very simple function that computes this task: 

```c++
char* mixedUppercase(char* source){
    for( int i=0; i< strlen(source); i+=2 ){
        // check if the character at the ith position is lowercase, otherwise skip
        if (*(source + i) >= 97 && *(source + i) <= 122){
            *(source + i) -= 32;
        }
    }

    return source;
}
```

This function takes a character array as input and performs the odd-uppercase replacement in place. This function returns the pointer to the first character of the original char vector.

### Test harness creation
First of all I'll start writing the TestHarness after I've determined what the expectations are for this function. First, I'll include the DeepState library's required header files: 
```c++
#include <deepstate/DeepState.hpp>
```

Then, using the TEST macro, I'll begin constructing the test function, taking into account the expected outcome from the `mixedUppercase` function.
The following code sample illustrates one possible implementation: 

```c++
TEST(MixedUppercase, OnlyGeneratedMixedUppercase) {
    char* str = DeepState_CStrUpToLen(50, "abcdefABCDEF");
    ASSUME_GT(strlen(str), 1);
    mixedUppercase(str);

    ASSERT_TRUE(isMixedUppercase(str))
        << str << " is not a mixed uppercase string";
}
```

This test function creates a string with a length of up to 50 characters, containing characters from the set `"abcdefABCDEF"`. 
After that we make the assumption that the the strings that are relevant for the test are the one of a length greter than 1.
Finally we execute our function and we check using the `ASSERT_TRUE` postcondition that the result is a Mixed-Uppercase string.

Where the `isMixedUppercase` function returns True if the string passed is a Mixed-Uppercase string, otherwise false.
```c++
bool isMixedUppercase(char* str){
    bool res = true;
    for (int i=0; i< strlen(str); i+=2){
        res = res && (*(str + i) >= 65 && *(str + i) <= 90);
    }
    return  res;
}
```
### Fuzz testing
Now that the Test harness is written, I can move on and perform some fuzz testing over the function. First I will compile the Test harness by specifying the linker to include the deepstate header with the `-ldeepstate` parameter
```bash
g++ -ldeepstate -o harness harness.cpp 
```

Finally I run a non deterministic fuzz testing on the TestHarness by executing the compiled TestHarness with the following parameters
```bash
./harness --fuzz --timeout=1 --input_which_test MixedUppercase_OnlyGeneratedMixedUppercase
```
### Result
After the execution I can check the results. As expected, in my case no error was found. 
```
INFO: Starting fuzzing
WARNING: No seed provided; using 1652815282
INFO: Done fuzzing! Ran 18918 tests (18918 tests/second) with 0 failed/17447 passed/1471 abandoned tests
```