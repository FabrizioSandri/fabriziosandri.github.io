---
toc: false
layout: post
description: Description of the steps that I will follow this week to improve the GitHub action's execution time
badges: true
tags: [Google Summer of Code, GitHub Action]
comments: false
title: GitHub action's execution time
related_posts: false
---

## Introduction
In the previous week I successfully integrated the original Docker action with a composite one. The main result of this update is a more flexible action that makes possible to publish the analysis results on Github; this is possible by publishing an artifact file containing the inputs and adding comments to pull requests. In the past working week (Week 7) I discussed with my mentor Randy about this change and he suggested me to prebuild the entire action and upload it to Docker Hub, which is a service provided by Docker for sharing container images[^1]. In this blog post I describe the process that will be followed this week, in order to prebuild the Docker image and publish it to Docker Hub.

## Considerations
The actual execution time of the Action is about 15-30 minutes. A huge amount of this time is spent installing all the missing dependencies, which includes the R package, devtools, and so on. Installing each of these every time the action is triggered is useless. A better approach instead would be to prebuild the docker container and save the image so that it can be reused each time the action will be run. This can be accomplished by publishing the image on Docker Hub. What I expect from this improvement is a reduction of the amount of time spent installing the dependencies.

One last thing to think about is moving the devtools installation command from the `entrypoint.sh` file to the Dockerfile so that it will be installed when the container is built. 

## Switch to a prebuilt docker image
The main idea is to replace the actual Action's Dockerfile with a simple composite step that runs the image from Docker Hub. 

In the actual implementation, every time the Action is triggered, a container is built in accordance with the Dockerfile, downloading each dependency one at a time, as you can see in the Dockerfile of the following sample. 
```dockerfile
FROM ubuntu:latest

# setup zoneinfo
RUN ln -snf /usr/share/zoneinfo/$INPUT_ZONEINFO /etc/localtime && echo $INPUT_ZONEINFO > /etc/timezone

### Dependencies installation
RUN apt update
RUN apt install -y build-essential gcc-multilib g++-multilib cmake \
                   python3-setuptools python3-dev libffi-dev clang valgrind \
                   libcurl4-gnutls-dev libxml2-dev libssl-dev wget \
                   libfontconfig1-dev libharfbuzz-dev libfribidi-dev \
                   libfreetype6-dev libpng-dev libtiff5-dev libjpeg-dev r-base

# Copy the files to the root filesystem of the container
COPY src/entrypoint.sh /entrypoint.sh
COPY src/analyze_package.R /analyze_package.R
RUN chmod +x /entrypoint.sh

# Executes `entrypoint.sh` when the Docker container starts up
ENTRYPOINT ["/entrypoint.sh"]
```

The new composite step will make use of the prebuilt image made available on Docker Hub. The dependencies are all installed in this image, so setting them up won't take any time; the only overhead will be caused by downloading the prebuilt action. This is the `action.yml` before the change.

```yaml
...
runs:
  using: "composite"
  steps:

    - uses: actions/checkout@v2 
      with:
        repository: FabrizioSandri/RcppDeepState-action
        path: RcppDeepState-action

    - name: Analyze the package with RcppDeepState 
      uses: ./RcppDeepState-action/docker
      with:
        fail_ci_if_error: ${{ inputs.fail_ci_if_error }}
        location: ${{ inputs.location }}
        seed: ${{ inputs.seed }}
        time_limit: ${{ inputs.time_limit }}
        max_inputs: ${{ inputs.max_inputs }}
...
```

After the change, it will look like this:

```yaml
...
runs:
  using: "composite"
  steps:

    - name: Analyze the package with RcppDeepState 
      uses: docker://fabriziosandri/rcppdeepstate:latest
      with:
        fail_ci_if_error: ${{ inputs.fail_ci_if_error }}
        location: ${{ inputs.location }}
        seed: ${{ inputs.seed }}
        time_limit: ${{ inputs.time_limit }}
        max_inputs: ${{ inputs.max_inputs }}
...
```
As you can see the `checkout` step has been removed, in fact there's no need to download the Dockerfile and the related files(`/docker` folder). This solves the problem of specifying the branch name in the checkout step mentioned by Randy in the pull request #4[^3]. However this introduces a new problem: I have to decide which event should trigger the update of the image on Docker Hub. A possible solution for this problem is to update the image every time a new commit on the master branch occurs. The problem with this solution is that, if I make a pull request and want to test some new updates made on the docker image, I have to push the result on the master branch to trigger the update on Docker Hub. 


## Continuous deploy of the Docker image
Each change to the Docker architecture must be reflected to the image published on Docker Hub. In order to make this process automatic and avoid to manually push the docker image to Docker Hub, as suggested by Randy, there is an existing GitHub Action that automatically builds and publishes Docker images to Docker Hub[^2]. This action can be integrated in the RcppDeepState-action repository inside a workflow's job that is triggered every time a commit on the master branch occurs. 

```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:

      - name: Checkout
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to DockerHub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
    
      - name: Build and push
        uses: docker/build-push-action@v3
        with:
          context: ./docker
          push: true
          tags: fabriziosandri/rcppdeepstate:latest
```

A token must be generated in order to publish the image to Docker Hub and authenticate the user before pushing the image on it. This token can be simply generated by going in Profile Settings > Security > Access Tokens > New Access Token. A pop up will appear, asking for the permissions that will be assigned to this token; the value "Read, Write, Delete" is sufficient. 

In order to prevent information theft, the token and the username should be kept as a repository secret. In the example above, the variables for the username and token are referred to as `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` respectively. These parameters have been set in the Security > Secrets > Actions menu on the Settings tab of the repository.
Once they are defined, they can be used in the following way` ${{ secrets.VARIABLE_NAME }}`.

<hr />

[^1]: [Docker Hub](https://hub.docker.com/)
[^2]: [GitHub Action - Build and push Docker images](https://github.com/marketplace/actions/build-and-push-docker-images)
[^3]: [Composite action #4](https://github.com/FabrizioSandri/RcppDeepState-action/pull/4#issuecomment-1183670955)