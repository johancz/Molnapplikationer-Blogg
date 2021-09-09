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
Continuous Integration (CI) is a set of principle and collection of practices to make development easier for teams working on the same code, faster and more reliable. CI tools help developers set up automated jobs such as building the code and running tests. Jobs can be setup to run when certain events are triggered, such as when a commit is pushed. If we use a build job as an example, it will build the code and should it fail it will prevent the developer from breaking the builds for all other developers working on the same code base. On a failure developers can be notified and if using Github Actions and pull requests, the jobs will show as having failed.

## What is a CI Pipeline?
A CI/CD (Continuous Integration/Continuous Deployment) Pipeline is a series of steps which need to executed in order release a new version of an app or other software. Some of the steps are; building, code reviewing, testing, further testing and finally deployment.

## This weeks assignment
[The assignment][The-Assignment] this week was to fork an older project and implement a Github Action CI Pipeline. The pipeline should trigger on every commit for all branches.

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

## Sources


{::comment}
Links
{:/comment}
[The-Assignment][https://pgbsnh20.github.io/PGBSNH20-molnapplikationer/cloud-lectures/ci#%C3%B6vningsuppgifter-i-grupp]
[Spacepark-Spaceinvaders-Original][]
