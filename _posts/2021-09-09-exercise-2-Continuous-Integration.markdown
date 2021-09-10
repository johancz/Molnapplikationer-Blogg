---
layout: post
title:  "Exercise 2 Continuous Integration"
date:   2021-09-09 20:50:00 +0200
# published: false
# hidden: true
# categories: blog
tags: Continuous-Integration, GitHub-Actions, CI, CI-Pipelines
---

## What is Continuous Integration (CI)?
Continuous Integration (CI) is a set of principle or collection of practices to make development easier for teams working on the same code, as well as faster and more reliable. This is done by frequently pushing code (the code can be partially complete as long as it doesn't break anything) and utilizing CI tools which aid developers in seting up automated jobs such as building the code and running tests. Jobs can be setup to run when certain events are triggered, such as when a commit is pushed. If we use a build job as an example, it will build the code and should it fail it will notify developers and if using Github Actions and pull requests, the workflows/jobs will show as having failed.

## What is a CI Pipeline?
A CI/CD (Continuous Integration/Continuous Deployment) Pipeline is a series of steps which need to executed in order release a new version of an app or other software. Some of the steps are; building, code reviewing, testing, further testing and finally deployment when all workflows/jobs and tests pass, as well as approval in code reviews.

## This weeks assignment
[The assignment][The-Assignment] this week was to [fork][Spacepark-Spaceinvaders-Fork] an [older project][Spacepark-Spaceinvaders-Original] and implement a Github Action CI Pipeline. The pipeline should trigger on every commit for all branches.


### How did I implement a CI pipeline (step-by-step)?

#### Steps:
##### #1:
I began by forking an old project ([Spacepark Spaceinvaders][Spacepark-Spaceinvaders-Original]).

##### #2:
Once the fork hade been created I clicked the "Actions" tab on the fork's github page, and selected one of the available .NET workflow templates (see the image below).

![.NET workflow template](/Molnapplikationer-Blogg/data/images/exercise-2-continuous-integration/Github-Actions-select-dotnet-workflow-template.png)

##### #3:
I committed the file (`/.github/workflows/dotnet.yml`) which was provided by the template to a new branch and opened a pull requests to merge this branch into the main branch. (The resulting pull request can be seen in the image below.)

![The resulting pull request](/Molnapplikationer-Blogg/data/images/exercise-2-continuous-integration/Github-PR-for-implementing-the-dotnet-workflow.png)

##### #4:
Since I committed the .net workflow as is the action fails at this point. Thus, the next step was to fix this by creating build jobs for all of the .NET-project in the repository.

##### #5
The fix was simple enough, by setting the default working-directory `./Source` the build command will run in the correct directory (see image).

![Github Action - working-directory](/Molnapplikationer-Blogg/data/images/exercise-2-continuous-integration/Github-Actions-dotnet-workflow-set-working-directory.png)

##### #6
Now that the action can build the .NET solution successfully, we move on the new issue. Because some of the unit tests in the test-project require access to a local database, they will fail. We solve this by temporarily disabling these tests. We commit the changes to our branch which triggers a new run.

![Pull request - no conflicts and all tasks were successful](/Molnapplikationer-Blogg/data/images/exercise-2-continuous-integration/Github-PR-no-conflicts-all-tasks-sucessful.png)

We now have a workflow which builds and tests our .NET projects, and we can merge the pull request.


## My GitHub Actions workflow Yaml-file:

[Link to the file in my repository][GitHub-Actions-Yaml-file]

```yaml
# Give the workflow a name
name: .NET Continuous Integration (CI) pipeline for building and testing

# Set the default working directory for all "run" steps on this workflow. In this case, commands like `dotnet build --no-restore` are executed in the `./Source` directory.
defaults:
  run:
    working-directory: ./Source

# Set the workflow to trigger on every push on all branches.
on:
  push:

jobs:
  # Here we create a "build" job.
  build:

    # The type of machine or environment to run the job on/in.
    runs-on: ubuntu-latest

    # Here we define the tasks (or steps) which execute various actions in sequence.
    steps:
    - uses: actions/checkout@v2 # This checks-out the repository so that the workflow can access it. 
    - name: Setup .NET
      uses: actions/setup-dotnet@v1 # This installs dotnet in the environment.
      with:
        dotnet-version: 5.0.x # Install dotnet 5.0.x
    - name: Restore dependencies
      run: dotnet restore # Restore any tools or dependencies for all projects in the solution.
    - name: Build
      run: dotnet build --no-restore # Build all projects without restoring.
    - name: Test
      run: dotnet test --no-build --verbosity normal # Run all tests in all projects, without building.
```


## Sources & Links
- [Continuous Integration (CI) Explained - semaphoreci.com][semaphoreci-dotcom-continuous-integration]
- [What is a CI/CD pipeline? - redhat.com][redhat-dotcom-what-is-cicd-pipeline]
- [CI Pipelines with GitHub Actions - redhat.com][redhat-dotcom-what-is-cicd-pipeline]
- [Why We Need Continuous Integration - semaphoreci.com][semaphoreci-dotcom-Why-We-Need-Continuous-Integration]
- [Microsoft's documentation for the dotnet CLI "build" command.][Microsoft-Docs-dotnet-build]
- [Workflow syntax for GitHub Actions][GitHub-Docs-Actions-Default-Run]
- [The original repository which we forked][Spacepark-Spaceinvaders-Original]
- [The fork][Spacepark-Spaceinvaders-Fork]
- [Events that trigger workflows - GitHub's documentation][GitHub-Docs-Actions-workflow-events]
- [My Yaml file][GitHub-Actions-Yaml-file]
- [Understanding GitHub Actions - GitHub's documentation][GitHub-Docs-Actions-understanding-github-actions]



[The-Assignment]: https://pgbsnh20.github.io/PGBSNH20-molnapplikationer/cloud-lectures/ci#%C3%B6vningsuppgifter-i-grupp
[Spacepark-Spaceinvaders-Original]: https://github.com/PGBSNH20/spacepark-spaceinvaders
[Spacepark-Spaceinvaders-Fork]: https://github.com/johancz/spacepark-spaceinvaders
[GitHub-Actions-Yaml-file]: https://github.com/johancz/spacepark-spaceinvaders/blob/main/.github/workflows/dotnet.yml
[GitHub-Docs-Actions-Default-Run]: https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#defaultsrun
[Microsoft-Docs-dotnet-build]: https://docs.microsoft.com/en-us/dotnet/core/tools/dotnet-build
[GitHub-Docs-Actions-workflow-events]: https://docs.github.com/en/actions/reference/events-that-trigger-workflows
[semaphoreci-dotcom-continuous-integration]: https://semaphoreci.com/continuous-integration
[redhat-dotcom-what-is-cicd-pipeline]: https://www.redhat.com/en/topics/devops/what-cicd-pipeline
[semaphoreci-dotcom-Why-We-Need-Continuous-Integration]: https://semaphoreci.com/community/tutorials/continuous-integration
[GitHub-Docs-Actions-understanding-github-actions]: https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions
