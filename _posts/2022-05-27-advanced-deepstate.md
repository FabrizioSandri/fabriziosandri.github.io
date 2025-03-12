---
toc: false
layout: post
description: An advanced fuzz testing description using DeepState in conjunction with Valgrind
badges: true
tags: [Google Summer of Code, DeepState, Valgrind]
comments: false
title: Advanced fuzz testing with DeepState and Valgrind
related_posts: false
---

# Advanced fuzz testing
In the previous posts we have seen how to use DeepState to find simple errors in programs. The problem of the basic fuzz testing approach is that it does not allow us to detect more subtle issues, such as memory faults.
In this article we introduce and explain a more advanced fuzz testing approach using the Valgrind memcheck tool and DeepState

## Valgrind introduction
Valgrind is a debugging and profiling tool for Linux applications. The Valgrind distribution comes with a set of useful tools: a thread error detector (Helgrind), a cache profiler (Cachegrind), a heap profiler(Massif), a memory problem analyzer (Memcheck), and other tools are included in the Valgrind suite [^1]. For our purpose and interest we will focus uniquely on the Memcheck tool. This tool allows you to do an advanced memory analysis to identify two types of memory errors: 
* Memory leaks
* Memory errors

The first is related to missed frees: a software allocates memory in the heap dynamically and then forgets to free it. The latter is caused by more subtle errors, such as reading or writing from uninitialized memory locations: a program writes some bytes in a memory location that has not been initialized. Valgrind has a module called Memcheck that can help you find these kinds of issues. 


### Usage
With some simple steps we can perform an advanced memory analysis over a compiled binary. Before the analysis can take place, in order to have as much information as possible from the analysis results such as the error line number, it's convenient to compile the program by passing the `-g` parameter to  the `g++` compiler. This option will produce more debugging information that will be included in the resulting binary. 

### Sample usage
The following is a description of the analysis procedure for a basic C++ program that allocates but does not release memory bytes. The program is reported in the following code snippet. As you can see the malloc invocation isn't followed by a free, thus what we are expecting from Valgrind is a warning from the Memcheck tool about the missing free. 

```c++
#include <cstdlib>

int main(int argc, char** argv){
    // allocates some space for 5 integers
    int* sample = (int*) malloc(sizeof(int) * 5);

    return 0;

}
```

As previously noted, the program should be constructed using the `-g` argument to provide greater information about any mistakes that may occur during the analysis. Another useful option is `--leak-check=full` which performs a full analysis for memory leak problems at the end of the execution.
```bash
g++ -o program -g program.cpp
```

Now it is possible to run Valgrind by specifying the tool to use with the `--tool` parameter
```bash
valgrind --tool=memcheck ./program
```
The result for this analysis will look similar to the followng one

```log
==108488== Memcheck, a memory error detector
==108488== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==108488== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==108488== Command: ./program
==108488== 
==108488== 
==108488== HEAP SUMMARY:
==108488==     in use at exit: 20 bytes in 1 blocks
==108488==   total heap usage: 2 allocs, 1 frees, 72,724 bytes allocated
==108488== 
==108488== 20 bytes in 1 blocks are definitely lost in loss record 1 of 1
==108488==    at 0x4845888: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==108488==    by 0x109151: main (program.cpp:5)
==108488== 
==108488== LEAK SUMMARY:
==108488==    definitely lost: 20 bytes in 1 blocks
==108488==    indirectly lost: 0 bytes in 0 blocks
==108488==      possibly lost: 0 bytes in 0 blocks
==108488==    still reachable: 0 bytes in 0 blocks
==108488==         suppressed: 0 bytes in 0 blocks
==108488== 
==108488== For lists of detected and suppressed errors, rerun with: -s
==108488== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```
The error summary shows that there is an error, in particular if we look at the leak summary we can see that 20 bytes are lost. So the question seems quite straightforward but, where does these 20 bytes come from? If we read carefully the valgrind description the error is found in the main function of our code at line `5` as stated by the following message
> `==108488==    by 0x109151: main (program.cpp:5)`

The error in fact comes from the call to the `malloc` function that allocates `5 * sizeof(int) = 5 * 4 bytes = 20` contiguous bytes in heap but doesn't free them. Note that the size of an integer is compiler and architecture dependent: in my case the size of a single int in memory is 4 bytes. 

## DeepState and Valgrind
Let's take a closer look at how Valgrind may be used in conjunction with the fuzzing approaches described in my earlier postings. As previously stated, basic fuzz testing is restricted in its ability to detect just certain types of programming problems. Consider combining the power of Valgrind memory checks with the benefits of fuzz testing. That is precisely what we will explore in this section.

Fuzz testing programs to find memory safety errors using Valgrind and DeepState allows to increase the probability of finding subtle errors.


### Use case example

Assume you're writing a function that accepts a pointer to the beginning of a string, copies it into a new heap-allocated string, and then frees it. Consider that while developing this function, the developer forgot about the size of the source string. A careful reader would have caught the mistake and addressed the problem, however we want to understand how Valgrind with the auxiliary of DeepState could quickly find the problem. As you may have seen, the issue is caused by two factors: 
* the contiguous memory area pointed by `newStr` can contain a string of at most 20 characters;
* the `memcpy` function doesn't check for any terminating null character.

This means that if someone passes as input parameter a string longer than 20 character, an heap overflow occur [^2].

```c++
#include <cstdlib>
#include <cstring>

void copyAndFreeString(char* source){
    // allocates some space for 20 characters
    char* newStr = (char*) malloc(sizeof(char) * 20);

    memcpy(newStr, source, strlen(source));
    free(newStr);
}

int main(int argc, char** argv){

    copyAndFreeString(argv[1]);

    return 0;
}
```

#### Test with Valgrind
Now let's consider to use Valgrind to check for any memory issue in the above program. The developer can try a lot of different options, forgetting about the border case(string greater than 20).
```bash
valgrind ./program 12345678
valgrind ./program hello world
valgrind ./program test
...
```

The result for all this test looks similar to the following for the first command execution.

```log
==9019== Memcheck, a memory error detector
==9019== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==9019== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==9019== Command: ./program 12345678
==9019== 
==9019== 
==9019== HEAP SUMMARY:
==9019==     in use at exit: 0 bytes in 0 blocks
==9019==   total heap usage: 2 allocs, 2 frees, 72,724 bytes allocated
==9019== 
==9019== All heap blocks were freed -- no leaks are possible
==9019== 
==9019== For lists of detected and suppressed errors, rerun with: -s
==9019== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```
As you can see, it seems like everything is working fine, no error, no leak.
However the developer missed to execute the border case. What we can learn here is that the greater the number of executions with varied inputs, the greater the chance of discovering subtle errors.

#### Test with Valgrind and DeepState
Let's integrate DeepState with the previous code fragment. First of all we remove the main function and we create a `TEST` function. This is the Test Harness creation phase, discussed in the previous posts[^3].

```c++
#include <cstdlib>
#include <cstring>

void copyAndFreeString(char* source){
    // allocates some space for 20 characters
    char* newStr = (char*) malloc(sizeof(char) * 20);

    memcpy(newStr, source, strlen(source));
    free(newStr);
}

TEST(StringCopy, CopyAndFreeString) {
    char* str = DeepState_CStrUpToLen(30, "abcdefABCDEF");

    ASSUME_GT(strlen(str), 1);
    copyAndFreeString(str);
}
```
As you can see the same standard structure is used to define a test function with the `TEST` macro: we start by providing some symbolic variables, then the pre conditions and finally the post conditions. The post conditions aren't useful in this situation because we're simply looking for memory related issues.

The compilation process can be started with the following command after specifying the compiler to include the headers for DeepState and to include debugging information in the final binary
```bash
g++ -ldeepstate -o program -g program.cpp
```

This time instead of running Valgrind and DeepState separated we can run them together to achieve the maximum performance

```bash
valgrind --tool=memcheck ./program --fuzz --timeout=1
```

```log
==16724== Memcheck, a memory error detector
==16724== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==16724== Using Valgrind-3.19.0 and LibVEX; rerun with -h for copyright info
==16724== Command: ./program --fuzz --timeout=1
==16724== 
INFO: Starting fuzzing
WARNING: No seed provided; using 1653599264
WARNING: No test specified, defaulting to first test defined (StringCopy_CopyAndFreeString)
==16724== Invalid write of size 8
==16724==    at 0x484FA11: memmove (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==16724==    by 0x114D25: copyAndFreeString(char*) (program.cpp:11)
==16724==    by 0x114E6B: DeepState_Test_StringCopy_CopyAndFreeString() (program.cpp:26)
==16724==    by 0x114D3D: DeepState_Run_StringCopy_CopyAndFreeString() (program.cpp:23)
==16724==    by 0x10FD10: DeepState_RunTestNoFork (in /home/fabri/test/testHarness/TestHarnessSample/program)
==16724==    by 0x110234: DeepState_FuzzOneTestCase (in /home/fabri/test/testHarness/TestHarnessSample/program)
==16724==    by 0x1104C7: DeepState_Fuzz (in /home/fabri/test/testHarness/TestHarnessSample/program)
==16724==    by 0x10B98A: main (in /home/fabri/test/testHarness/TestHarnessSample/program)
==16724==  Address 0x4df6cf0 is 16 bytes inside a block of size 20 alloc'd
==16724==    at 0x4845888: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==16724==    by 0x114CFF: copyAndFreeString(char*) (program.cpp:9)
==16724==    by 0x114E6B: DeepState_Test_StringCopy_CopyAndFreeString() (program.cpp:26)
==16724==    by 0x114D3D: DeepState_Run_StringCopy_CopyAndFreeString() (program.cpp:23)
==16724==    by 0x10FD10: DeepState_RunTestNoFork (in /home/fabri/test/testHarness/TestHarnessSample/program)
==16724==    by 0x110234: DeepState_FuzzOneTestCase (in /home/fabri/test/testHarness/TestHarnessSample/program)
==16724==    by 0x1104C7: DeepState_Fuzz (in /home/fabri/test/testHarness/TestHarnessSample/program)
==16724==    by 0x10B98A: main (in /home/fabri/test/testHarness/TestHarnessSample/program)

    ... truncated file ...

INFO: Done fuzzing! Ran 1 tests (1 tests/second) with 0 failed/1 passed/0 abandoned tests
==16724== 
==16724== HEAP SUMMARY:
==16724==     in use at exit: 0 bytes in 0 blocks
==16724==   total heap usage: 3 allocs, 3 frees, 72,754 bytes allocated
==16724== 
==16724== All heap blocks were freed -- no leaks are possible
==16724== 
==16724== For lists of detected and suppressed errors, rerun with: -s
==16724== ERROR SUMMARY: 6 errors from 3 contexts (suppressed: 0 from 0)
```

This time the results are quite different: Valgrind with the auxiliary of DeepState was able to discover 6 errors from 3 different contexts. As we can see from the analysis of Valgrind (I have truncated the analysis to only the first error) the error is caused by the `memmove` invocated by `memcpy` in the program ad line `11`.

## Conclusion
Using only one tool is not always the best approach to find subtle programming errors. In fact in this article we discussed a advanced technique to perform a more sophisticated analysis. By combining fuzz testing from DeepState with memory analysis from Valgrind we were able to address more errors than the one we could have discovered by using a single technique. 



[^1]: [Valgrind tools](https://valgrind.org/info/tools.html)
[^2]: [Heap overflow](https://cwe.mitre.org/data/definitions/122.html)
[^3]: [Test Harness creation guide](https://fabriziosandri.github.io/gsoc-2022-blog/deepstate/fuzz/c++/2022/05/25/about-deepstate.html)