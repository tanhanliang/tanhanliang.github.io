---
layout:		post
title: 		Docker Cheat Sheet
date: 		2018-08-13
categories:	docker
---

## What is this
This page contains docker commands that I found useful.

## Build an image
~~~shell
docker build [OPTIONS] PATH | URL | -
~~~

| Flag | Meaning |
| \-\-tag, -t | Tag image for easy reference. Name automatically generated if no tag specified. |
{: .tablelines}


Example (assumes Dockerfile in current directory)
~~~shell
docker build -t pytorch .
~~~

## Run an image
This creates a new container from an image. This command is not meant for running an existing, stopped container.
Simplified form:
~~~shell
docker run [OPTIONS] <image name>
~~~

| Flag | Meaning |
| -it | Start an interactive session (e.g bash). |
| \-\-name | Assign a name to the resulting image. | 
| -p | Publish port or range of ports from container to host. |
| \-\-runtime | Allows Docker to run with another 'engine', for example one that is compatible with Nvidia GPUs (requires installation of nvidia-docker). | 
| \-\-user, -u | Run the container as a particular user. Can specify a UID or username, for example `--user root` or `--user 0`.
| \-\-volume, -v | Bind a folder on host to a folder in the container. |
{: .tablelines}

A separate container has to be created if you want to run as root.

Example
~~~shell
docker run \
    -it \
    --volume ~/Desktop/code:/home/thldocker/code \
    --runtime=nvidia \
    -p 8888:8888 \
    --name pytorch \
    pytorch
~~~

## Run existing container
~~~shell
docker start <container-name>
docker attach <container-name>
~~~

## See all containers
~~~shell
docker ps -a
~~~

## Remove all containers
~~~shell
docker rm (docker ps -a -q)
~~~

## See all images
~~~shell
docker images
~~~

## Remove all images
~~~shell
docker rmi (docker images -a -q)
~~~

## Push image to Docker Hub
Log in to https://hub.docker.com/ and create a repository. Next, log in to Docker Hub from the terminal:
~~~shell
docker login --username=dockerhub-username
~~~

Tag the desired image. Form:
~~~shell
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
~~~

The `TARGET_IMAGE` is of the form `<dockerhub-username>/<image-name>`. The tag should help you understand what the image is for.

For example:
~~~shell
docker tag pytorch thldocker/pytorch:ml
~~~

Finally, push to Docker Hub:
~~~shell
docker push <dockerhub-username>/<image-name>
~~~

## References
1. <a href="https://docs.docker.com/engine/reference/commandline/build/#parent-command">Docker build</a>
2. <a href="https://docs.docker.com/engine/reference/run/">Docker run</a>
3. <a href="https://docs.docker.com/engine/reference/commandline/start/">Docker start</a>
4. <a href="https://docs.docker.com/engine/reference/commandline/tag/#description">Docker tag</a>
5. <a href="https://docs.docker.com/engine/reference/commandline/attach/">Docker attach</a>

<style>
.tablelines table, .tablelines td, .tablelines th {
        border: 1px solid black;
        }
</style>
