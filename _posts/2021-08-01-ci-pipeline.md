---
title: "CI Pipeline"
excerpt: "Infrastructure to run tasks locally and on a CI server"
categories:
  - Blog
tags:
  - Jenkins
---

# Introduction

There are a number of tasks you may wish to carry out in a repository. For example

- Compiling source code
- Packaging binaries
- Generating documentation / diagrams
- Run the test suite
- Running checks
  - Static analysis
  - Coverage

# Desire for ease

Sometimes the steps required to performing these tasks can get quite complicated. If the tasks are not easy for a developer, then

- The developer may not bother with certain tasks. Which means they may commit changes without running all the checking tools, or inspecting the generated documentation.
- They may invoke the tools incorrectly by passing the wrong flags or omitting files.

# Desire to run locally

Having all the tasks run on the server is good, however, it is all desirable to have every task run locally too. The benefits are

- It reduces the reliance on servers, which means developers can work offline.
- It provides developers with quicker feedback because they can run each stage individually (and usually faster) without having to wait until they push to remote (or create a pull request)
- It help maintainability and the development of new stages because each stage can be tested locally first.
- Keeps pull request history clean. Without being able to run stages locally then usually the first few commits are the author fixing errors the CI toolchain picked up on.

# Tools to run locally

A task runner should be used to run the tasks. This allows

- The developer to easily see the list of tasks which can be run.
- The developer to ignore the details of how each tool is invoked.

The most ubiquitous task runner would be Make. Using this as an example, in the makefile there would be a target for `test-coverage` which defines what needs to happen to generate the coverage report. The user simply invokes `make test-coverage`
to generate the report without worrying about how it is generated.

Depending on your language and build system you may choose to use a different tool such as CMake (add_custom_target) or Python Invoke.

# Continuous integration server

It is essential all the tasks run on a continuous integration server such as Jenkins. This ensures every pull request does not unknowingly introduce bad changes.  
The task runner provides a simple interface for invoking each tool. Therefore the Jenkinsfile can be a simple wrapper that calls into the task runner in the same manner as the developer does. For example

```
stage('build') {
    steps {
        sh "invoke build"
```

With this approach the developer and server are both invoking the tools in the same manner. The definition of which is mastered in the task runner code.

![local-server](/assets/2021-08-01-ci-pipeline/local-server.drawio.svg)

# Dependency versioning

Using the task runner we have ensured that each developer and the server will invoke the tools in the correct manner. However if they are not running the same versions of the tools they may see different results.

Usually every repository will contain a file describing it's dependencies (requirements.txt, debian/control, etc). This file allows the developers and server to ensure they are using the correct version. However it can get messy when the developers or server are required to build different projects which require different versions of the same dependency. Some programming languages like Python provide solutions to this such as using pipenv.

A more generic solution is to use containers to provide an isolated working environment with the correct dependencies.

# Docker container

A Dockerfile in the repository is used to instruct Docker how to create an image with all the necessary tools. The developers and the server can invoke all tasks from within the container to ensure they all get the same result.

The benefit are

- It helps new starts because there is no lengthy installation process. Simply start the container are you are up and running.
- It makes teams more autonomous. If they want to change tooling versioning they can do so by editing their dockerfile which they have ownership of. Rather than having to coordinate with other teams to change the version a shared Jenkins server.

Development dockerfiles will have these main steps

1. Start from a base image.

```
FROM ubuntu:20.04
```

2. Copy the file listing the dependencies

```
COPY ./debian/control /tmp/vplane-config-qos/debian/control
```

3. Install the dependencies from the file

```
RUN mk-build-deps \
 && apt install --yes --fix-missing ./vplane-config-qos-build-deps_*_all.deb
```

# Docker locally

1. To build the image from the dockerfile

```
docker image build --tag project_name .
```

2. When running the container there are a few considerations.  
   **Command line:** We have to pass the input from the host to the input of the container (`--interactive`) and tell the process the input is coming from a terminal (`--tty`)  
   **Shared data:** A mounted drive gives the container access to to the repository and give the host access to generated artifacts (`--mount`)  
   **User_ID:** By default the ubuntu container has root UID which means all the generated artifacts will be owned by root. Use the host's UID in the container (`--user`)

```
docker run --interactive --tty --mount type=bind,src=${PWD},dst=/tmp/vplane-config-qos --user $(id -u):$(id -g) project_name
```

3. Invoke the task runner from within the container

```
I have no name!@b403fd3a236c:/$ invoke build
```

Note: `I have no name!` is because there is no matching UID in the /etc/passwd file.

# Docker on server

Customize the Jenkins execution environment by instructing Jenkins to use the dockerfile checked into the repository to build the container.

```
pipeline {
    agent {
       dockerfile true
```

Jenkins will add a shared volume and set the working directory. Jenkins will also set the UID & GID to the same as the host. So there is no need to explicitly specify these arguments.

TODO: Limit cpus/memory
