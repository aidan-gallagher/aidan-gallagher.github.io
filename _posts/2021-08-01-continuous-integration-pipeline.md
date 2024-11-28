---
title: "Continuous Integration Pipeline"
excerpt: "Using a container and task runner to run tasks locally and on a continuous integration server."
categories:
  - Articles
tags:
  - Jenkins
  - Docker
  - Infrastructure
  - Devops
---

## Introduction

There are a number of tasks a developer may wish to perform such as

- Compiling source code
- Packaging binaries
- Generating documentation / diagrams
- Running the test suite
- Running checks
  - Static analysis
  - Coverage

## Issues With Complicated Tasks

Sometimes the steps required to performing these tasks can get quite complicated. If the tasks are not easy for a developer

- The developer may not bother with certain tasks. Which means they may commit changes without running all the checking tools, or inspecting the generated documentation.
- They may invoke the tools incorrectly by passing the wrong flags or omitting files.

## Benefits of Running Tasks Locally

Having all the tasks run on the server is great, however, it is desirable to be able to run every task locally too. The benefits are

- It reduces the reliance on servers, which means developers can work offline.
- It provides developers with quicker feedback because they can run each stage individually (and usually faster) without having to wait until they push to the server (or create a pull request).
- It help maintainability and the development of new stages because each stage can be tested locally first.
- It keeps pull request history clean because the author can discover and fix their warnings offline rather than updating their PR to fix warnings.

## Using a Task Runner to Simplify Execution

A task runner should be used to run the tasks. This allows the developer

- To easily see the list of tasks which can be run.
- To ignore the details of how each tool is invoked.

The most ubiquitous task runner is GNU Make. Using it as an example, in the makefile there would be a target for `test-coverage` which defines what needs to happen to generate the coverage report. The user simply invokes `make test-coverage` to generate the report without worrying about how it is generated.

Depending on the projects main programming language and build system you may choose to use a different tool such as CMake (with add_custom_target) or Python Invoke.

## Consistent Tool Invocation on CI Server

It is essential all the tasks run on a continuous integration (CI) server such as Jenkins. This ensures every pull request does not unknowingly introduce bad changes.

The task runner provides a simple interface for invoking each tool. Therefore the Jenkinsfile can be a simple wrapper that calls into the task runner in the same manner as the developer does. For example

```
stage('build') {
    steps {
        sh "make build"
```

With this approach the developer and server are both invoking the tools in the same manner. The definition of which is mastered in the task runner code.

![ci-pipeline](/assets/2021-08-01-continuous-integration-pipeline/ci-pipeline.drawio.svg)

## Consistent Tool Versioning

Using the task runner we have ensured that each developer and the server will invoke the tools in the correct manner. However if they are not running the same versions of the tools they may see different results.

Usually every repository will contain a file describing it's dependencies (requirements.txt, debian/control, etc). This file allows the developers and server to ensure they are using the correct version. However it can get messy when the developers or server are required to build different projects which require different versions of the same dependency. Some programming languages like Python provide solutions to this such as using pipenv.

A more generic solution is to use containers to provide an isolated working environment with the correct dependencies.

## Using Docker to Ensure Consistent Tool Versioning

A Dockerfile in the repository is used to instruct Docker how to create an image with all the necessary tools. The developers and the server can invoke all tasks from within the container to ensure they all get the same result.

![ci-pipeline-with-docker](/assets/2021-08-01-continuous-integration-pipeline/ci-pipeline-with-container.drawio.svg)

The benefit are

- It helps new starts because there is no lengthy installation process. Simply start the container are you are up and running.
- It makes teams more autonomous. If they want to change tooling versioning they can do so by editing their dockerfile which they have ownership of rather than having to coordinate with other teams to change the version on a shared Jenkins server.

Development dockerfiles will have these main steps

1. Start from a base image

   ```
   FROM ubuntu:20.04
   ```

2. Copy the file listing the dependencies

   ```
   COPY ./debian/control /tmp/my-project/debian/control                    # Debian example

   COPY ./dev-requirements.txt /tmp/my-project/dev-requirements.txt        # Pip example
   ```

3. Install the dependencies from the file

   ```
   RUN apt-get install --yes devscripts equivs \
    && mk-build-deps --install --remove --tool='apt-get --yes'             # Debian example

   RUN apt-get install --yes python3-pip \
    && pip3 install --requirement dev-requirements.txt                     # Pip example
   ```

## Running Docker Locally

### Using the CLI

1. To build the image from the dockerfile

   ```
   docker image build --tag project_name .
   ```

2. To run the container

   ```
   docker run --interactive --tty --mount type=bind,src=${PWD},dst=/tmp/my-project --user $(id -u):$(id -g) project_name
   ```

   - **Command line:** We have to pass the input from the host to the input of the container (`--interactive`) and tell the process the input is coming from a terminal (`--tty`).
   - **Shared data:** A mounted drive gives the container access to to the repository and gives the host access to generated artifacts (`--mount`).
   - **User_ID:** By default the ubuntu container has root UID which means all the generated artifacts will be owned by root. Use the host's UID in the container (`--user`).

3. Invoke the task runner from within the container

   ```
   I have no name!@b403fd3a236c:/$ invoke build
   ```

   Note: `I have no name!` is because there is no matching UID in the /etc/passwd file.

### Using VSCode
1. Open the project in VSCode
2. F1 -> Dev Containers: Rebuild and Reopen folder in Container
3. Terminal -> New Terminal (which will now be in container)
4. Invoke the task runner from within the container


## Running Docker on CI Server

Customize the Jenkins execution environment by instructing Jenkins to use the dockerfile checked into the repository to build the container.

```
pipeline {
    agent {
        dockerfile true
```

There is no need to explicitly specify any args because Jenkins will

- Add a shared volume.
- Set the working directory.
- Set the UID & GID to the same as the host.

## Only Checking Modified Files

Most checking tools (flake8, mypy, etc) can be run over a directory and they will check every relevant file. Sometimes instead of checking every file you may only want to check files that have been modified.
This is useful when

- Working on a legacy system where you want to introduce new rules going forward.
- The checks take a long time.

Using git a list of modified files can be determined

```
git diff -G'.' --diff-filter=rd --find-renames=100% --name-only --commits {TARGET-BRANCH}...{SOURCE-BRANCH}
```

- `-G"."`: ignores files with only permission changes
- `--diff-filter=rd`: ignores files that have been renamed or deleted
- `--find-renames=100%`: ensures files classified as renamed have the exact same contents
- `--name-only`: only print file names

An example of a task (defined in python) that checks flake8 on modified files would look like this

```
def flake8(commits="master...HEAD"):
    files = get_files(commits)
    python_files = get_files_by_types(files, ["Python"])
    if python_files:
        subprocess.run(f"flake8 --count {python_files}", shell=True)
```

## Summary

- Developers often have several tasks to perform in a repository.
- The tasks should be easy to run locally.
- A task runner provides an interface for developers and build servers to invoke tasks in the same manner.
- Containers allow
  - Developers and servers to share the same environment to reduce inconsistences.
  - New starts to get up and running quickly.
  - Dependency installations to be isolated from the host OS.
- Git can be used to only check modified files.

## Example
A minimal HelloWorld example that implements these ideas can be found on [github](https://github.com/aidan-gallagher/helloworld-debian-package/tree/ci-pipeline-example).