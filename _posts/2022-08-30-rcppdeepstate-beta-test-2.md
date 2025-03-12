---
toc: false
layout: post
description: Solution for the problem that prevents the action from being run in remote pull requests. 
badges: true
tags: [Google Summer of Code, GitHub Action]
comments: false
title: Issue with RcppDeepState-action beta test 
related_posts: false
---


## Introduction to the issue
After performing beta tests on the binsegRcpp[^1] package using 
RcppDeepState-action I discovered two issues.

The first is that the user need consent from a single maintainer with write 
access before running a workflow within a remote repository. This has been
demonstrated with my mentor through pull request 
[tdhock/binsegRcpp#16](https://github.com/tdhock/binsegRcpp/pull/16).
In fact, this is explicitly stated in the GitHub documentation[^2]: 

> To help prevent this, workflows on pull requests to public repositories from 
some outside contributors will not run automatically, and might need to be 
approved first. By default, all first-time contributors require approval to run 
workflows.

However, this is not the only issue: the person who opens a pull request is not
permitted to use the remote repository's `GITHUB_TOKEN` to automatically add a 
comment in the pull request. The token is only valid inside the same repository. 
This is clearly explained in the GitHub docs[^3]: 

> When you enable GitHub Actions, GitHub installs a GitHub App on your 
repository. The GITHUB_TOKEN secret is a GitHub App installation access token. 
You can use the installation access token to authenticate on behalf of the 
GitHub App installed on your repository. The token's permissions are limited to
the repository that contains your workflow.

## Workaround
The primary purpose is to run RcppDeepState-action on CRAN packages available on
GitHub and share the findings with the package authors. To accomplish this task,
the following steps can be used as a workaround for the previous issues: 

1. fork the remote repository;
2. Create a self-pull request(from fork to fork) in the forked repository to 
examine the package using RcppDeepState-action;
3. If RcppDeepState finds at least one error, make a pull request on the remote 
repository(from fork to upstream);
4. Copy and paste the RcppDeepState report from the forked repository's pull 
request into the pull request of the remote repository. 

## Implementation
According to the script that automatically forks and opens a self pull request
written in the previous blog post[^4], we can write a script that, after the 
RcppDeepState-action workflow is done, opens a remote pull request on the
original repository and copies and pastes the RcppDeepState report within the
remote pull request.

The first step is to import the required libraries and get the GitHub personal 
access token (PAT) stored in an environment variable called GITHUB_PAT in my 
case. This token will be used to authorize the push of local commits to a 
repository’s remote branch. To generate this token, you can follow the 
instructions provided by GitHub[^5].

```R
library("gh")
library("git2r")
library("data.table")

cred <- cred_token(token = "GITHUB_PAT")
batch_size <- 10 # adjust the batch size
organization <- "RcppDeepState"
```

In this case, we have defined two additional parameters, one of which is the 
`batch_size` parameter, which limits the number of packages tested. This option
must be equivalent to the `batch_size` parameter used when executing the script
that forks and creates a pull request[^4], or else some pull request will not be
submitted. Furthermore, we define the `organization` parameter, which searches 
for forked repositories inside the organization provided in this parameter. This
parameter has been added after the problem outlined in the [Future work](https://fabriziosandri.github.io/gsoc-2022-blog/github%20action/2022/08/23/rcppdeepstate-beta-test.html#future-work) 
paragraph of the previous blog post. 

Given a `pkg.repos` data table produced by following the steps specified in 
Dr.Toby Dylan Hocking’s blog post[^6], the implementation of the automatic
process outlined above is described in the next steps.

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


```R
pull_body_template <- paste("This PR adds a new Github Action which runs",
                   "RcppDeepState+valgrind on your package. That means the C++",
                   "functions of your package will be tested with random",
                   "inputs, and there will be a comment like this one for each",
                   "new PR (which reports if valgrind found any issues with",
                   "random inputs).\n\n#### RcppDeepState analysis result\n",
                   "This package contains problems, according to",
                   "RcppDeepState. The report was generated by",
                   "[RcppDeepState-action](https://github.com/FabrizioSandri/RcppDeepState-action)",
                   "in this repository's fork and is accessible ")

pkg.repos <- nc::capture_first_df(pkg.repos, repo.url=list(
                                  "https://github.com/", repo_full_name=list(
                                  ".*/", repo_name=".*")))

for (repo_i in seq(min(batch_size, nrow(pkg.repos)))){
  repo <- pkg.repos[repo_i]
  fork_name <- paste(organization, repo$repo_name, sep="/")
  fork_pull_url <- paste("https://github.com", fork_name, "pull", "1", sep="/")

  # get the report comment from the forked repository
  fork_comments_endpoint <- paste0("GET /repos/", fork_name, 
                                   "/issues/1/comments")
  rcppdeepstate_pull <- gh(fork_comments_endpoint)
  if (length(rcppdeepstate_pull) == 0){
    warn_msg <- paste("The report comment has not been generated for", 
                      repo$repo_full_name)
    message(warn_msg)
  }else{
    rcppdeepstate_comment <- sub("<!-- RcppDeepState-action comment-->", "",
                                rcppdeepstate_pull[[1]]$body)

    if (grepl("No error has been reported", rcppdeepstate_comment, fixed = TRUE)){
      warn_msg <- paste("No error has been reported by RcppDeepState for the",
                        "package", repo$repo_full_name)
      message(warn_msg)
    }else{
      # get the remote repo default branch name
      upstream_endpoint <- paste0("GET /repos/", repo$repo_full_name)
      upstream_details <- gh(upstream_endpoint)

      # open a pull request in the original repository
      pulls_endpoint <- paste0("POST /repos/", repo$repo_full_name, "/pulls")
      pull_title <- "Analyze the package with RcppDeepState"
      pull_body <- paste0(pull_body_template,"[here](", fork_pull_url, ").")

      head <- paste(organization, "RcppDeepState", sep=":")
      pull_response <- gh(pulls_endpoint, title=pull_title, body=pull_body, 
                          owner=upstream_details$owner$login, repo=repo$repo_name, 
                          base=upstream_details$default_branch, head=head)

      # issue a comment with the rcppdeepstate report
      pull_number <- pull_response$number
      original_comments_endpoint <- paste0("POST /repos/", repo$repo_full_name, 
                                      "/issues/", pull_number, "/comments")
      gh(original_comments_endpoint, owner=upstream_details$owner$login, 
        repo=repo$repo_name, issue_number=pull_number, body=rcppdeepstate_comment)
    }
  }
}
```

## Test
Before we execute the aforementioned script, we must first run the previous blog
post's code[^4] to fork the original repository and create the RcppDeepState 
reports. Remember that the new repositories will be created within the 
organization specified in the relative variable, as described in the
[Future work](https://fabriziosandri.github.io/gsoc-2022-blog/github%20action/2022/08/23/rcppdeepstate-beta-test.html#future-work)
paragraph of the previous blog post[^4].

Following that, we can ensure that all reports for the repositories being
tested have been generated automatically using the following piece of code: 
```R
for (repo_i in seq(min(batch_size, nrow(pkg.repos)))){
  repo <- pkg.repos[repo_i]
  fork_name <- paste(organization, repo$repo_name, sep="/")

  fork_comments_endpoint <- paste0("GET /repos/", fork_name, 
                                   "/issues/1/comments")
  rcppdeepstate_pull <- gh(fork_comments_endpoint)
  
  if (length(rcppdeepstate_pull) < 1){
    warn_msg <- paste("The report for", repo$repo_full_name,
                      "has not yet been generated.")
    message(warn_msg)
  }
}
```

I tested this script an hour after the automatic fork procedure had finished 
with a batch size of `10`, and it reported that the report for two repositories 
had not been generated. 

```
The report comment has not been generated for ekstroem/MESS
The report comment has not been generated for MikeJaredS/hermiter
```

By analyzing the logs, I discovered that the reports were not generated because 
of a missing library that is not installed and is not specified in the package's 
prerequisites. According to the error message for the `hermiter` package, the
missing library is `RcppParallel`: 
```bash
fatal error: 'RcppParallel.h' file not found
#include <RcppParallel.h>
         ^~~~~~~~~~~~~~~~
```

### Run the remote pull request script
Running the script that submits a pull request remotely will result in the 
following output: 
```R
> source("remote_pull.R")
No error has been reported by RcppDeepState for the package shabbychef/BWStest
The report comment has not been generated for ekstroem/MESS
The report comment has not been generated for MikeJaredS/hermiter
```
Aside from the two previously mentioned repositories, the remote pull request 
for shabbychef/BWStest will not be submitted because RcppDeepState found no 
errors. Instead, the report will be submitted in a matter of seconds to the
following repositories: 
* ironholds/humaniformat [pull request #8](https://github.com/Ironholds/humaniformat/pull/8)
* jMotif/jmotif-R [pull request #35](https://github.com/jMotif/jmotif-R/pull/35)
* Ironholds/olctools [pull request #6](https://github.com/Ironholds/olctools/pull/6)
* WinVector/RcppDynProg [pull request #1](https://github.com/WinVector/RcppDynProg/pull/1)
* CollinErickson/CGGP [pull request #44](https://github.com/CollinErickson/CGGP/pull/44)
* paulhibbing/PAutilities [pull request #8](https://github.com/paulhibbing/PAutilities/pull/8)
* ms609/Quartet [pull request #63](https://github.com/ms609/Quartet/pull/63)

## Conclusion and Future work
With this article demonstrated how to add another step to the script that 
automatically forks and opens a pull request to the remote repositories. In this
manner, I used a workaround to address the issues described in the article's 
introduction. 

Future work will involve increasing the `batch_size` option to the
maximum supported value in order to detect issues in all of the packages 
identified by Akhila. 

<hr />

[^1]: [binsegRcpp repository](https://github.com/tdhock/binsegRcpp)
[^2]: [Approving workflow runs from public forks](https://docs.github.com/en/actions/managing-workflow-runs/approving-workflow-runs-from-public-forks)
[^3]: [Automatic token authentication](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)
[^4]: [Beta test RcppDeepState-action on GitHub hosted CRAN packages](https://fabriziosandri.github.io/gsoc-2022-blog/github%20action/2022/08/23/rcppdeepstate-beta-test.html)
[^5]: [Creating a personal access token](https://docs.github.com/en/enterprise-server@3.4/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
[^6]: [R packages on github](https://tdhock.github.io/blog/2022/packages-on-github/)
