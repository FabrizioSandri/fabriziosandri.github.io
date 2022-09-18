---
toc: false
layout: post
description: A quick introduction to DeepState fuzzing library 
badges: true
tags: [DeepState]
comments: false
title: DeepState introduction
---

# DeepState Introduction
DeepState is a framework which provides developers the possibility to perform symbolic unit testing on some particular C++ functions by writing test harnesses. For writing a test harness, Deepstate follows a similar approch to the one of Google tests [^1].
The common problem between most of the symbolic unit testing libraries is that you as developer have to know how to use other binary analysis tools. With Deepstate this problem is solved by provinding a common interface the test harness creation process.


## Installation
The installation process is quite simple, you just need to follow the istruction provided in the _Readme_ file in the Deepstate repository on GitHub [^2]. In my case after the installation of all the necessary dependencies, I downloaded the repository snapshot using the standard `git clone` approach

```bash
git clone https://github.com/trailofbits/deepstate.git
```

and then started the compilation process with the auxiliary of _Make_ tools

```bash
mkdir deepstate/build && cd deepstate/build
cmake ../
make
```

If the compilation process succeed the next step is the installation of the compiled files in the proper location in the host system:

```bash
sudo make install
```

## Test Harness creation
Testing C++ code is not a straightforward task and it need to be accomplished in the correct way, otherwise it will lead to a failure both in the test and also in the source code. A Test Harness is a collection of software that allows to test a particular program by executing under some conditions.

The first step in test harness creation is to include  the library and optionally set the namespace, in order to use all the functionalities provided by DeepState :
```c++
#include <deepstate/DeepState.hpp>
using namespace deepstate;
```

A Test Harness definition in DeepState starts with the `TEST` macro. This macro takes two arguments as input:
* a unit name
* a test name

The body of the function that starts with `TEST` will contain the following parts:
* symbols definition
* pre-conditions
* post-conditions

Let's dive into this three main parts of a Test Harness creation procedure.

### Symbols definition
The first part consists in the definition of the symbols that will be used as input for the function we need to test. The main goal for this task is to have a set of variables that will be filled by the DeepState with some non-deterministic data.

Symbolic execution tools and fuzzers needs to know which variable is symbolic in order to control them. Symbols definitions for basic data types (`int`, `char`, ..) are defined by concatenating the data type name with the `symbolic_` prefix (`symbolic_int`, `symbolic_char`, ..). 

DeepState provides us some useful functions to initialize our symbols with random data:
* `int DeepState_IntInRange(int low, int high)` : initializes the symbol with an integer in the range low-high
* `float DeepState_FloatInRange(float low, float high)` : initializes the symbol with a float number in the range low-high
* `double DeepState_DoubleInRange(double low, double high)` : initializes the symbol with a double number in the range low-high
* `char* DeepState_CStr_C(size_t len, const char* allowed)` : initializes a character array (a string)  of length `len` with a set of allowed character taken from the character array `allowed
* `char* DeepState_CStrUpToLen(size_t maxLen, const char* allowed)` : initializes a character array (a string) of length  up to `len` with a set of allowed character taken from the character array `allowed`

### Pre conditions
Sometimes there is the necessity to constraint a little bit more the symbolic variables defined previously. This means that you have some particular constraint that your symbolic variable need to comply with, before running into the tests. All the tests that do not pass the pre conditions will be considered `abandoned`.

Pre conditions are defined using the `ASSUME` macro and also by the following more specialized ones:
* `ASSUME_EQ` : checks for the equality of the 2 parameters (operator ==)
* `ASSUME_NE` : checks if the 2 parameters are not equal (operator !=)
* `ASSUME_LT` : checks if the first parameter is less than the second (operator <)
* `ASSUME_LE` : checks if the first parameter is less or equal to the second (operator <=)
* `ASSUME_GT` : checks if the first parameter is greater than the second (operator >)
* `ASSUME_GE` : checks if the first parameter is greater or equal to the second (operator >=)

#### Example 
Suppose you have to pass to your test only strings with a minimum length of 5 characters and up to 10 characters (both inclusive). This types of strings can not be created by only using `DeepState_CStrUpToLen`. We have to constraint the fact that the length of the string should also be greater then 5. This may be accomplished by using a precondition that checks if the created string meets the requirement that its length be larger than or equal to 5. With the preceding list in mind, this can be implemented with the `ASSUME_GE` macro in the following way:

```c++
TEST(UnitName, TestName) {

    char* str = DeepState_CStrUpToLen(50, "abcdefABCDEF");
    ASSUME_GE(strlen(str), 5);
    
    ...
}
```

### Post conditions
The last part is the check of the execution result of a test. This can be done with post conditions that checks if the results satisfy some constraints. Similar to the pre conditions, the post condition defines a list of macros that help us to perform this checks. This time a post condition is defined using the `ASSERT` macro and all it's specialized versions (`ASSERT_EQ`, `ASSERT_NE`, `ASSERT_FALSE`, ...).

#### Example 
Assume that we want to test a function that compress a string as input into a fixed length string : an hash function like MD5. A possible post condition will check if the hashed string is of a certain length (in the case of md5, exactly 32 characters).

```c++
TEST(UnitName, TestName) {
    char* str = DeepState_CStrUpToLen(50, "abcdefABCDEF");

    // Pre conditions
    ASSUME_GE(strlen(str), 5);

    // execution
    char* hashedString = md5(str);

    // post conditions
    ASSERT_EQ(strlen(hashedString), 32);
}
```

### Test harness compilation
You can compile the test harness using a standard C++ compiler like g++ by specifying the linker to include the deepstate header with the `-ldeepstate` parameter
```bash
g++ -ldeepstate -o harness harness.cpp 
```

The output of this procedure is a compiled harness file that can be executed.

## Fuzz test execution
The compiled test harness can be executed using some analysis tools like `manticore`, `angr` or `valgrind` to discover memory leaks in the code. You can also run it in a standalone mode by executing it directly in the console with the `--fuzz` option. This last option enables DeepState to perform brute force fuzzing. Another useful option is `--timeout` that allows to specify a timeout(in seconds) after which the fuzzing procedure will be stopped. In addition with the `--input_which_test` you can specify the test to execute in the format `UnitName_TestName`.

```bash
./harness --fuzz --timeout=5 --input_which_test UnitName_TestName
```

This execution will give us the total number of _failed_, _passed_ and _abandoned_ tests

## Advanced fuzz testing
I'll go through how to use DeepState in conjunction with more complex tools like Valgrind in the next blog posts.


[^1]: [https://google.github.io/googletest/](https://google.github.io/googletest/)
[^2]: [https://github.com/trailofbits/deepstate#buildnrun](https://github.com/trailofbits/deepstate#buildnrun)