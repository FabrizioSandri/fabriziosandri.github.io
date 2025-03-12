---
toc: false
layout: post
description: Guide to write a custom test harness for a function that cannot be analyzed because of some datatypes falling outside of the supported ones.
badges: true
tags: [Google Summer of Code, RcppDeepState]
comments: false
title: Write a custom test harness for functions with unsupported datatypes
related_posts: false
---


## Introduction
The list of supported datatypes maintained by RcppDeepState is used to automatically create test harnesses for functions whose supported datatypes fall within this list. This list was created using a frequency table of the top 100 Rcpp types found in CRAN packages[^1]. The complete list of supported datatypes is provided below: 
* `int`
* `double`
* `string`
* `Rcpp::NumericVector`
* `Rcpp::IntegerVector`
* `Rcpp::NumericMatrix`
* `Rcpp::CharacterVector`
* `arma::mat`

It is possible for a function you want to analyze to have a parameter that is not on this list, in which case RcppDeepState will notify you that the function cannot be examined due to an unsupported parameter. 
```
We can't test the function - unsupported_datatype - due to the following datatypes falling out of the allowed ones: LogicalVector

Failed to create testharness for 1 functions in the package - unsupported_datatype
```

This just indicates that the automatic procedure to create a test harness cannot be used, not that your function cannot be analyzed at all. The test harness must therefore be manually created together with its directory structure, and it must then be examined.
I'll explain this procedure in this blog article. 

## The procedure
RcppDeepState involves two steps:
* *generator*: automatically generates the harness and all the inputs 
* *runner*: analyze the package running each function given the inputs of the generator step

The generator step cannot be used for a function whose inputs are not supported, so we must manually run the steps to construct the directory structure for this function. This can be done using the existing RcppDeepState functions. 

Given the path of the package being analyzed `package_location` and the name of the function which datatypes fall out of the supported ones `function_name`, the `deepstate_create_makefile` function can be used to generate the directory structure for the custom test harness:
```R
deepstate_create_makefile(package_location, function_name)
```

If everything goes smoothly, you can begin creating the custom test harness for your function. The test harness file must be named `function_name_DeepState_TestHarness.cpp`, where `function_name`is the name of the function you want to analyze. You can use the following template as a reference. 
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

Remember to replace `<package_name>` in the `TEST(<package_name>, generator)` definition with  your package name (without angular brackets). 

Once the harness has been created you can generate the inputs and analyze the function containing unsupported datatypes as arguments.
```R
deepstate_fuzz_fun(package_location, function_name)
result <- deepstate_harness_analyze_pkg(package_location)
```

## Example
I want to provide an example of the `unsupported_datatype` function that can be found in the testSAN package[^2]. This function uses a `LogicalVector` as its single parameter, thus RcppDeepState will skip this function from the analysis.

To solve this I followed all the steps mentioned above. First of all I ran the `deepstate_create_makefile` function, remembering that I had to define the missing variables(`package_location` and `function_name`):
```R
package_location <- "/home/fabri/test/testHarness/RcppDeepState/inst/testpkgs/testSAN"
function_name <- "unsupported_datatype"

deepstate_create_makefile(package_location, function_name)
```

At the end of this, a directory containing the following content has been created:
```
unsupported_datatype
├── inputs
├── Makefile
└── unsupported_datatype_output
```
Then I moved on and created the test harness file inside the `unsupported_datatype` directory and I renamed this file as `unsupported_datatype_DeepState_TestHarness.cpp`. I followed the template provided above. This is the result:
```c++
#include <fstream>
#include <RInside.h>
#include <iostream>
#include <RcppDeepState.h>
#include <qs.h>
#include <DeepState.hpp>

RInside Rinstance;

/** FUNCTION SIGNATURE */
int unsupported_datatype(Rcpp::LogicalVector param);

/** INPUTS */ 
#define INPUTS \
  LogicalVector param = RcppDeepState_LogicalVector();

Rcpp::LogicalVector RcppDeepState_LogicalVector(){
  int rand_size = DeepState_IntInRange(1,20);
  Rcpp::LogicalVector rand_vec(rand_size);
  for(int i = 0 ; i < rand_size;i++){      
    rand_vec[i] = DeepState_IntInRange(0,1);  
  }

  return rand_vec;
}

TEST(testSAN, generator){
  INPUTS  
}

TEST(testSAN, runner){
  INPUTS

  /** INPUTS DUMP */
  qs::c_qsave(param, "./inputs/param.qs", "high", "zstd", 1, 15, true, 1);

  try{
    /** FUNCTION INVOCATION */
    unsupported_datatype(param);
  }catch(Rcpp::exception& e){
    std::cout<<"Exception Handled"<<std::endl;
  }
}
```

You can notice that I have created an auxiliary function `RcppDeepState_LogicalVector` used to generate a random `LogicalVector` of size ranging from 1 to 20 elements. 

Finally I run the last two steps:
```R
deepstate_fuzz_fun(package_location, function_name)
result <- deepstate_harness_analyze_pkg(package_location)
```

If you print the `result` table you will find that `unsupported_datatype` has finally been tested with RcppDeepState.

<hr />

[^1]: [Top 100 Rcpp types found in CRAN packages](https://github.com/FabrizioSandri/RcppDeepState/issues/10#issuecomment-1179190239)
[^2]: [unsupported_datatype function](https://github.com/FabrizioSandri/RcppDeepState/blob/master/inst/testpkgs/testSAN/src/unsupported_datatype.cpp)