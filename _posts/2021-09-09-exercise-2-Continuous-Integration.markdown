---
layout: post
title:  "Exercise 2 Continuous Integration"
date:   2021-09-09 20:50:00 +0200
published: false
# hidden: true
# categories: blog
tags: Continuous-Integration, GitHub-Actions, CI, CI-Pipelines
---

## What is Continuous Integration (CI)?
Continuous Integration (CI) is a set of principle and collection of practices to make development easier for teams working on the same code, faster and more reliable. CI tools help developers set up automated jobs such as building the code and running tests. Jobs can be setup to run when certain events are triggered, such as when a commit is pushed. If we use a build job as an example, it will build the code and should it fail it will prevent the developer from breaking the builds for all other developers working on the same code base. On a failure developers can be notified and if using Github Actions and pull requests, the jobs will show as having failed.


## What is a CI Pipeline?
A CI/CD (Continuous Integration/Continuous Deployment) Pipeline is a series of steps which need to executed in order release a new version of an app or other software. Some of the steps are; building, code reviewing, testing, further testing and finally deployment.


