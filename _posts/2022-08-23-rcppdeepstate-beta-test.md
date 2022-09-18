---
toc: false
layout: post
description: RcppDeepState-action beta test on other packages
badges: true
tags: [GitHub Action]
comments: false
title: Beta test RcppDeepState-action on GitHub hosted CRAN packages 
---


## Introduction
My mentor has assigned me an interesting task to begin working on this week: 
test the current RcppDeepState GitHub Action on GitHub-hosted Rcpp-based 
packages. Akhila, the previous RcppDeepState maintainer, had already conducted 
a similar task, however this was done locally rather than in the package 
repository. Here's a list of all the [Rcpp-based packages where RcppDeepState found issues](https://akhikolla.github.io./packages-folders/)[^1]. 

Now that RcppDeepState has been integrated with GitHub action, we can test more 
packages stored on GitHub; all we have to do is fork the original repositories, 
setup the action within them, and then make a pull request so that the action 
returns a comment with the analysis result. The main advantages of using 
RcppDeepState-action inside a repository are that it allows to: 
* dynamically check for issues inside packages using continuous integration; 
* reduce the risk of code level bugs that can compromise the entire package; 
* improve the quality of the final package by making it easier to detect subtler
bugs, receiving quick feedbacks and alerts if an error is detected;


## The problem
At the time of writing, CRAN lists 18493 packages, 312 of which have problems, 
according to Akhila's report[^1]. Given this list of packages, the question is 
whether it is feasible to determine if a package is hosted on GitHub. If so, 
can RcppDeepState be run on this repository? 

To begin with, the answer to the first question is yes, and my mentor 
Dr.Toby Dylan Hocking supplied me with a fantastic method that allows me to 
find the GitHub repository of a package by evaluating the package's metadata 
available on CRAN.
This method is based on locating the `https://github.com` prefix inside the 
metadata of the package. More information on this technique is available in the
corresponding blog post[^2]. 


The answer to the second question is affirmative; once we have a link to the 
repository, we can simply fork it, initialize RcppDeepState-action using the 
`RcppDeepState::ci_setup()` method, and submit a pull request; RcppDeepState's 
report will be displayed as a comment within the pull request. The problem here 
is that, given the number of packages listed in the previous step, doing these 
steps one by one is impractical; consequently, in this post, I propose a method 
for automating this process. 

## Solution
The solution makes use of four libraries: 
* *gh*[^3]: a minimal client to access the REST API of GitHub;
* *git2r*[^4]: an interface to the `libgit2` library, which provides access to 
Git repositories with some basic commands;
* *data.table*[^5]: a library to aggregate large data and run fast 
operations;
* *RcppDeepState*[^6]: a package to fuzz test your R library's C++ code in order 
to find more subtle bugs like memory leaks or even more general memory errors.

### Steps 
The first step is to import the required libraries and get the GitHub personal 
access token (PAT) stored in an environment variable called `GITHUB_PAT` in my 
case. This token will be used to authorize the push of local commits to a 
repository's remote branch. To generate this token, you can follow the 
instructions provided by GitHub[^7]. 

```R
library("gh")
library("git2r")
library("RcppDeepState")
library("data.table")

cred <- cred_token(token = "GITHUB_PAT")
```

Given a `pkg.repos` data table produced by following the steps specified in 
Dr.Toby Dylan Hocking's blog post[^2], the implementation of the automatic 
fork/pull-request process mentioned above is described in the next steps. 

```R
> pkg.repos
          Package                                  repo.url
  1: humaniformat https://github.com/ironholds/humaniformat
  2:       jmotif        https://github.com/jMotif/jmotif-R
  3:     olctools     https://github.com/Ironholds/olctools
  4:  RcppDynProg  https://github.com/WinVector/RcppDynProg
  5:      BWStest     https://github.com/shabbychef/BWStest
 ---                                                       
111:       tweenr       https://github.com/thomasp85/tweenr
112:         uwot        https://github.com/jlmelville/uwot
113:       vapour       https://github.com/hypertidy/vapour
114:           wk         https://github.com/paleolimbot/wk
115:      wkutils    https://github.com/paleolimbot/wkutils
```

We begin by removing the `http://github.com` prefix from each repository url, 
resulting in with a new column containing strings in the format 
`<repository owner>/<repository name>`. 
```R
pkg.repos[, repo_full_name := sub("https://github.com/", "", repo.url) ]
```

```R
> pkg.repos
          Package                                  repo.url         repo_full_name
  1: humaniformat https://github.com/ironholds/humaniformat ironholds/humaniformat
  2:       jmotif        https://github.com/jMotif/jmotif-R        jMotif/jmotif-R
  3:     olctools     https://github.com/Ironholds/olctools     Ironholds/olctools
  4:  RcppDynProg  https://github.com/WinVector/RcppDynProg  WinVector/RcppDynProg
  5:      BWStest     https://github.com/shabbychef/BWStest     shabbychef/BWStest
 ---                                                                              
111:       tweenr       https://github.com/thomasp85/tweenr       thomasp85/tweenr
112:         uwot        https://github.com/jlmelville/uwot        jlmelville/uwot
113:       vapour       https://github.com/hypertidy/vapour       hypertidy/vapour
114:           wk         https://github.com/paleolimbot/wk         paleolimbot/wk
115:      wkutils    https://github.com/paleolimbot/wkutils    paleolimbot/wkutils

```

Then we can iterate over the repositories listed above, forking and cloning each
one. Let us call each repository in the loop `repo_full_name`.
```R
fork_endpoint <- paste0("POST /repos/", repo_full_name, "/forks")
fork_result <- gh(fork_endpoint)

repo <- clone(fork_result$clone_url, fork_result$name)
config(repo, http.followRedirects='true')
```

After successfully cloning the repository, a new branch for the RcppDeepState 
analysis can be created. `RcppDeepState` is the name of this new branch. 
```R
test_branch_name <- "RcppDeepState"
test_branch <- branch_create(last_commit(repo), test_branch_name)
checkout(repo, test_branch_name)
```

The following step is to determine if the repository includes a legitimate 
package. This is accomplished by checking the existence of the `DESCRIPTION` 
file within the repository's root: if this file exists, the repository includes 
a valid package that can be examined using RcppDeepState; otherwise, 
RcppDeepState cannot analyze the package. 
```R
if (!file.exists(file.path("./", fork_result$name, "DESCRIPTION"))){
  stop("The repository doesn't contain a valid package")
}
```

We can now use the existing `ci_setup` function to initialize the workflow file 
within the repository. This function accepts as input the location of the 
repository on the filesystem and a list of parameters corresponding to the 
action's inputs. In this scenario, we've specified `fail_ci_if_error=TRUE` to 
cause the CI process to fail if an error is discovered, and `comment=TRUE` to 
print the report comment inside the pull request that will be produced in the 
following phase. 
```R
RcppDeepState::ci_setup(fork_result$name, fail_ci_if_error=TRUE, comment=TRUE)
```

The last step is to push the new changes to the forked repository and submit a 
pull request.
```R
# commit and push the workflow file
add(repo, file.path("./", fork_result$name, ".github", "workflows", "*"))
commit(repo, message="RcppDeepState CI Setup")
push(repo, "origin", paste("refs", "heads", test_branch_name, sep="/"),
    credentials=cred)

# open the pull request
pulls_endpoint <- paste0("POST /repos/", fork_result$full_name, "/pulls")
pull_title <- "Analyze the package with RcppDeepState"
pull_body <- paste("### RcppDeepState Analysis\nThis pull request aims to find", 
                   "bugs in this R package using RcppDeepState-action")
gh(pulls_endpoint, title=pull_title, owner=fork_result$owner$login,
    repo=fork_result$name, body=pull_body, base=fork_result$default_branch,
    head=test_branch_name)
```

### Final script
We get the following script by combining all of the previous steps with the 
solution supplied by my mentor. As you can see, a `batch_size` option has been 
introduced to allow you to choose the number of repositories to test. This 
option was added to prevent the creation of 115 repositories within your GitHub 
account. 

```R
library("gh")
library("git2r")
library("RcppDeepState")
library("data.table")

cred <- cred_token(token = "GITHUB_PAT")
batch_size <- 2 # adjust the batch size

if(!file.exists("problems.html")){
  download.file(
    "https://akhikolla.github.io./packages-folders/",
    "problems.html")
}
prob.dt <- nc::capture_all_str(
  "problems.html",
  '<li><a href="',
  Package=".*?",
  '[.]html">')

if(!file.exists("packages.rds")){
  download.file(
    "https://cloud.r-project.org/web/packages/packages.rds",
    "packages.rds")
}
meta.mat <- readRDS("packages.rds")
meta.dt <- data.table(meta.mat)
meta.prob <- meta.dt[prob.dt, on="Package"]

pkg.repos <- meta.prob[, nc::capture_all_str(
  c("",URL), # to avoid attempting to download URL.
  repo.url="https://github.com/.*?/[^#/ ,]+"),
  by=Package]

pkg.repos[, repo_full_name := sub("https://github.com/", "", repo.url) ]

for (repo_full_name in head(pkg.repos$repo_full_name, batch_size)){
  
  fork_endpoint <- paste0("POST /repos/", repo_full_name, "/forks")
  fork_result <- gh(fork_endpoint)

  repo <- clone(fork_result$clone_url, fork_result$name)
  config(repo, http.followRedirects='true')

  test_branch_name <- "RcppDeepState"
  test_branch <- branch_create(last_commit(repo), test_branch_name)
  checkout(repo, test_branch_name)

  ### check if the repository's root contains a valid package
  if (!file.exists(file.path("./", fork_result$name, "DESCRIPTION"))){
    stop("The repository doesn't contain a valid package")
  }
  
  RcppDeepState::ci_setup(fork_result$name, fail_ci_if_error=TRUE,
                          comment=TRUE)

  # commit and push the workflow file
  add(repo, file.path("./", fork_result$name, ".github", "workflows", "*"))
  commit(repo, message="RcppDeepState CI Setup")
  push(repo, "origin", paste("refs", "heads", test_branch_name, sep="/"),
      credentials=cred)

  # submit a pull request  
  pulls_endpoint <- paste0("POST /repos/", fork_result$full_name, "/pulls")
  pull_title <- "Analyze the package with RcppDeepState"
  pull_body <- paste("### RcppDeepState Analysis\nThis pull request aims to", 
                    "find bugs in this R package using RcppDeepState-action")
  gh(pulls_endpoint, title=pull_title, owner=fork_result$owner$login,
      repo=fork_result$name, body=pull_body, base=fork_result$default_branch,
      head=test_branch_name)
}
```

## Test
Before running the preceding script, I assumed that running it with a 
`batch_size` of 115(`nrow(pkg.repos$repo_full_name)`) would result in a massive
generation of repositories within my GitHub account.
As a preliminary solution, I set the `batch_size` argument to `2`, which means
that just two packages will be examined. I explain a possible approach to avoid
this massive production of repositories under my GitHub user profile in the 
[Future work](#future-work) paragraph.

The following repositories will be tested with a `batch_size` of 2: 
* [ironholds/humaniformat](https://github.com/ironholds/humaniformat)
* [jMotif/jmotif-R](https://github.com/jMotif/jmotif-R)


Following the execution of the above script, two repositories will be 
automatically generated by forking the originals. If you go inside the 
repositories' pull requests, you'll see that a new pull request titled 
`Analyze the package with RcppDeepState` has been automatically submitted.
After a few minutes, when the CI checks are completed, you will see a comment 
inside the pull request with the analysis result. 

### Results
The findings of the analysis are publicly available on Github, and they 
highlight some issues within the evaluated packages. If we compare the results 
to those discovered by Akhila[^1], we can see that there are some similarities. 

Here are the links to the results:
* [humaniformat results](https://github.com/FabrizioSandri/humaniformat/pull/1)
* [jmotif-R results](https://github.com/FabrizioSandri/jmotif-R/pull/1)

## Future work
Creating 115 repositories, as previously said, will result in a huge generation
of repositories within my GitHub account. This is not a problem solely because 
of the number of repositories, but if I need to remove all of them, I will 
undoubtedly have to write a script to do so; this can be a dangerous task if 
done in my current working environment (my user profile) because I may
accidentally specify the incorrect condition and end up deleting the wrong
repositories. 

One possible solution is to create a Github Organization and instruct the above 
script to fork the repositories to a specific organization rather than to my 
GitHub account. This can be accomplished by passing an extra argument to the 
GitHub REST API: the `organization` parameter; this parameter must be set to the
organization's name. The rest of the code will be left unchanged. 

```R
fork_endpoint <- paste0("POST /repos/", repo_full_name, "/forks")
fork_result <- gh(fork_endpoint, organization="<org name>")
```


<hr />

[^1]: [Rcpp Packages with Issue Detected using RcppDeepState](https://akhikolla.github.io./packages-folders/)
[^2]: [R packages on github](https://tdhock.github.io/blog/2022/packages-on-github/)
[^3]: [CRAN package: gh](https://cran.r-project.org/web/packages/gh)
[^4]: [CRAN package: packages](https://cran.r-project.org/web/packages/git2r)
[^5]: [CRAN package: data.table](https://cran.r-project.org/web/packages/data.table)
[^6]: [RcppDeeepState](https://github.com/FabrizioSandri/RcppDeepState)
[^7]: [Creating a personal access token](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)