---
layout: post
title:  "Exercise 3 Containers and orchestration"
date:   2021-09-15 17:52
# published: false
# hidden: true
# categories: blog
tags: Dockerfile, docker, containers, container-orchestration, docker-compose, 
---


<!-- <p style="color: red; font-size: 5em;">Work in progress</p> -->


## What did you install locally / Github?

### Locally
- Github Desktop
- Docker Desktop
- Visual Studio Code
- Windows Terminal (Command Prompt)

### Github
- The Workflow template [".NET" (by Github Actions)][github-com-actions-starter-workflows-dotnet]


## How did I get the app to run in a (Docker) container?
1. I began by forking [Stephan's Repository][github-com-apprepo-original]
1. I then cloned [my forked repository][github-com-apprepo-myfork] to my computer with Github Desktop.
1. Next I wanted to test the app:
    ```shell
    dotnet publish -o ./publish # publish since this is the command I would use in my Github Actions workflow later
    dotnet run
    ```
    I could then open `http://localhost:5000` as well as `https://localhost:5001` to make sure everything was functioning correctly.
1. Now it was time to get it running in Docker container, so I [created a "Dockerfile".][github-com-apprepo-commit-dockerfile]
1. Of course I tested my new Docker setup by running something along the lines of:
    ```shell
    docker build -t simplewebhalloworldapp .
    docker run -d -p 8080:80 --name SimpleWebHalloWorldApp simplewebhalloworldapp
    ```
1. When I could confirm that my image/container functioned as expected, I [added a "docker-compose.yml" file.][github-com-apprepo-commit-dockercomposefile]
1. The app can now be started with `docker compose up`.

Et voila, the app runs in a container.


## My "Dockerfile":

If you'd rather read this in my github repo, [click here][github-com-apprepo-myfork-Dockerfile].

```yaml
# Specify that we want to base our image upon "dotnet sdk 3.1 image" in microsoft's container registry (MCR), and fetch it.
FROM mcr.microsoft.com/dotnet/sdk:3.1 AS build-env
# Set the working directory (i.e. the directory where we want our files and the directory in which commands (such as COPY, RUN and ENTRYPOINT) should be executed in)
WORKDIR /app

# Copy "SimpleWebHalloWorld.csproj" to our workdir and restore it in its own container layer.
COPY *.csproj ./
RUN dotnet restore

# Copy everything else to our workdir and build the project
COPY . ./
RUN dotnet publish -c Release -o out

# Build runtime image
FROM mcr.microsoft.com/dotnet/aspnet:3.1
# Set the working directory again.
WORKDIR /app
# Copy the built files to its own "out" directory.
COPY --from=build-env /app/out .
# Set the app's entry point by having docker invoke "dotnet SimpleWebHalloWorld.dll" in the command line.
ENTRYPOINT ["dotnet", "SimpleWebHalloWorld.dll"]
```


## My GitHub Pipeline.

My pipeline has one workflow, called ".NET", which contains two jobs. Both jobs run whenever something is pushed to the "master". branch or a pull_request event occurs on the "master" branch. The first job, called "build", is not necessary or relevant to this exercise, so we'll skip discussing that one.

The second job, "build-and-push-docker-image" runs on ubuntu, sets the working directory and then checks out the code with `- uses: actions/checkout@v2`.

It then uses the `docker/login-action@v1.10.0` action twice, first to log into (authenticate against) GitHub's Container Registry (GHCR) and finally to log into Docker's registry.
The addresses of the registries for the login processes are specified, as well as the the usernames and passwords (see images below).

![Workflow file - login action for Docker's registry](/Molnapplikationer-Blogg/data/images/exercise-3-containers-and-orchestration/github-repo-actions-workflow-login-action-github.png)


![Workflow file - login action for Docker's registry](/Molnapplikationer-Blogg/data/images/exercise-3-containers-and-orchestration/github-repo-actions-workflow-login-action-docker.png)

These usernames and passwords are not hardcoded since we don't want to expose that information publicly. Instead we use secrets, but more on that later.

Finally we build the image with the `docker/build-push-action` where we specify the registries we want to push our image to and set a "version tag" (e.g. "latest" or "9").

![Workflow file - login action for Docker's registry](/Molnapplikationer-Blogg/data/images/exercise-3-containers-and-orchestration/github-repo-actions-workflow-build-push-action.png)

`{% raw %}${{ github.run_number }}{% endraw %}` is a unique number representing the number of times this workflow has executed, a version if you will.

Here we tell our workflow to push our image to:
- GitHub Container Registry:
  - `ghcr.io/johancz/simplewebhalloworld` with the "tag": `latest`
  - `ghcr.io/johancz/simplewebhalloworld` 
  > where `ghcr.io` is the registry, `johancz` is the namespace/user and `simplewebhalloworld`.
- Docker Hub:
  - `docker.io/johancz/simplewebhalloworld` with the "tag": `latest`
  - `docker.io/johancz/simplewebhalloworld`
  > where `docker.io` is the registry, `johancz` is the namespace/user and `simplewebhalloworld`.


### My Workflow file:
If you'd prefer to read this in my github repo, [click here][github-com-apprepo-myfork-workflow-pipeline-file].

![Workflow file - login action for Github's registry](/Molnapplikationer-Blogg/data/images/exercise-3-containers-and-orchestration/github-repo-actions-workflow-file.png)

## How did I handle secrets?

### GitHub Container Repository
For GitHub's container registry (GHCR) I elected to use the token provided by github "GITHUB_TOKEN" as this is what [Docker recommends][github-com-docker-login-action-repo-authentication-ghcr]) in their login-action repository. The token is used in my workflow file here:

![Workflow file - login action for Github's registry](/Molnapplikationer-Blogg/data/images/exercise-3-containers-and-orchestration/github-repo-actions-workflow-login-action-github.png)

### Docker Hub
The [recommended (by Docker)][github-com-docker-login-action-repo-authentication-docker-hub] way of authenticating against Docker Hub is to create a [personal access token][docs-docker-com-managing-access-tokens] which I stored in my repository as secrets in my repository (see the image below).

![Github Repository Secrets - Docker Hub authentication info](/Molnapplikationer-Blogg/data/images/exercise-3-containers-and-orchestration/github-repo-secrets-for-docker-hub.png)

The tokens are used in my workflow file here:

![Workflow file - login action for Docker's registry](/Molnapplikationer-Blogg/data/images/exercise-3-containers-and-orchestration/github-repo-actions-workflow-login-action-docker.png)


<!-- ## Docker CLI authentication against repositories -->



<!-- ## How do pushes with `docker/build-push-action@v2` end up in the correct repository? -->



## Sources & Links
- [Dockerfile reference - docs.docker.com][docs-docker-com-dockerfile-reference]
- [Publishing Docker images - docs.github.com][docs-github-com-publish-docker-images]
- [".NET" starter Workflow - Github Actions][github-com-actions-starter-workflows-dotnet]
- [Docker build-push action repository and documentaiton][github-com-buildpush-action]
- [Getting Started With ASP.NET Core & Docker][Getting-Started-With-ASP.NET-Core-&-Docker]:
- [How YOU can Dockerize a .Net Core app][How-YOU-can-Dockerize-a-.Net-Core-app]
- [Get started with Docker Compose][Get-started-with-Docker-Compose]
- [A Practical Introduction to Docker Compose][A-Practical-Introduction-to-Docker-Compose]


[docs-docker-com-dockerfile-reference]: https://docs.docker.com/engine/reference/builder/
[docs-docker-com-managing-access-tokens]: https://docs.docker.com/docker-hub/access-tokens/
[docs-github-com-publish-docker-images]: https://docs.github.com/en/actions/guides/publishing-docker-images
[docs-github-com-actions-trigger-events-pull-request]: https://docs.github.com/en/actions/reference/events-that-trigger-workflows#pull_request
[github-com-buildpush-action]: https://github.com/docker/build-push-action
[github-com-actions-starter-workflows-dotnet]: https://github.com/actions/starter-workflows/blob/028df69d88fa6b986e3ec1f52b4ae52300e87c5a/ci/dotnet.yml
[github-com-apprepo-original]: https://github.com/skjohansen/SimpleWebHalloWorld
[github-com-apprepo-myfork]: https://github.com/johancz/SimpleWebHalloWorld
[github-com-apprepo-myfork-Dockerfile]: https://github.com/johancz/SimpleWebHalloWorld/blob/master/Dockerfile
[github-com-apprepo-myfork-workflow-pipeline-file]: https://github.com/johancz/SimpleWebHalloWorld/blob/master/.github/workflows/ci-pipeline.yml
[github-com-apprepo-commit-dockerfile]: https://github.com/johancz/SimpleWebHalloWorld/commit/f648502cec84ead0a9855640e064be5f83764f6b
[github-com-apprepo-commit-dockercomposefile]: https://github.com/johancz/SimpleWebHalloWorld/commit/ce9ff9935ab082203036151d61f8e387573d6ad7#diff-e45e45baeda1c1e73482975a664062aa56f20c03dd9d64a827aba57775bed0d3
[github-com-docker-login-action-repo-authentication-docker-hub]: https://github.com/docker/login-action#docker-hub
[github-com-docker-login-action-repo-authentication-ghcr]: https://github.com/docker/login-action#github-container-registry
[Getting-Started-With-ASP.NET-Core-&-Docker]: https://morioh.com/p/5414a74be39d
[How-YOU-can-Dockerize-a-.Net-Core-app]: https://softchris.github.io/pages/dotnet-dockerize.html
[Get-started-with-Docker-Compose]: https://docs.docker.com/compose/gettingstarted/
[A-Practical-Introduction-to-Docker-Compose]: https://hackernoon.com/practical-introduction-to-docker-compose-d34e79c4c2b6
