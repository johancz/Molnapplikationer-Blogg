---
layout: post
title:  "Exercise 3 Containers and orchestration"
date:   2021-09-14 21:15
# published: false
# hidden: true
# categories: blog
tags: Dockerfile, docker, containers, container-orchestration, docker-compose, 
---


<p style="color: red; font-size: 5em;">Work in progress</p>


## What did you install locally / Github?

### Locally
- Github Desktop
- Docker Desktop
- Visual Studio Code
- Windows Terminal (Command Prompt)

### Github
- The Workflow template [".NET" (by Github Actions)][github-com-actions-starter-workflows-dotnet]


## How did you get the app to run in a (Docker) container?
1. I began by forking [Stephan's Repository][github-com-apprepo-original]
1. I then cloned [my forked repository][github-com-apprepo-myfork] to my computer with Github Desktop.
1. Next I wanted to test the app:
    ```shell
    dotnet publish -o ./publish # publish since this is the command I would use in my Github Actions workflow later
    dotnet run
    ```
    and opened `http://localhost:5000` as well as `https://localhost:5001` to make sure everything was functioning correctly.
1. Now it was time to get it running in Docker container, so I [created a "Dockerfile".][github-com-apprepo-commit-dockerfile]
1. Of course I tested my new Docker setup by running something along the lines of:
    ```shell
    docker build -t simplewebhalloworldapp .
    docker run -d -p 8080:80 --name SimpleWebHalloWorldApp simplewebhalloworldapp
    ```
1. When I could confirm that my image/container functioned as expected, I [added a "docker-compose.yml" file.][github-com-apprepo-commit-dockerfile]



## The "Dockerfile"
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


## Sources & Links
- [Dockerfile reference - docs.docker.com][docs-docker-com-dockerfile-reference]
- [".NET" starter Workflow - Github Actions][github-com-actions-starter-workflows-dotnet]



[docs-docker-com-dockerfile-reference]: https://docs.docker.com/engine/reference/builder/
[github-com-actions-starter-workflows-dotnet]: https://github.com/actions/starter-workflows/blob/028df69d88fa6b986e3ec1f52b4ae52300e87c5a/ci/dotnet.yml
[github-com-apprepo-original]: https://github.com/skjohansen/SimpleWebHalloWorld
[github-com-apprepo-myfork]: https://github.com/johancz/SimpleWebHalloWorld
[github-com-apprepo-commit-dockerfile]: https://github.com/johancz/SimpleWebHalloWorld/commit/f648502cec84ead0a9855640e064be5f83764f6b
[github-com-apprepo-commit-dockercomposefile]: https://github.com/johancz/SimpleWebHalloWorld/commit/ce9ff9935ab082203036151d61f8e387573d6ad7
