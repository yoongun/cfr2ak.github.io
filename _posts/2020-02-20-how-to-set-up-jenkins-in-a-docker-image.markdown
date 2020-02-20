---
layout: post
title:  "How to set up jenkins in a docker image"
date:   2020-02-20
tags: [jenkins, docker]
---

{% include mathjax.html %}


# How to set up jenkins over docker

## Prerequisite

You need to know your host os' docker group's gid. To check the gid, enter the command below on the shell.

```bash
id
```

## Final Dockerfile

```dockerfile
FROM jenkins/jenkins:latest

ENV DOCKER_GROUP_GID <docker_gid>
USER root

RUN wget https://download.docker.com/linux/static/stable/x86_64/docker-18.03.1-ce.tgz
RUN tar -xvf docker-18.03.1-ce.tgz
RUN mv docker/* /usr/bin/
RUN groupadd -o -g ${DOCKER_GROUP_GID} docker 
RUN usermod -g docker jenkins

USER jenkins 
```

## Explanation

First, pull the jenkins official image.

```dockerfile
FROM jenkins/jenkins:latest
```

Then set the environment variable.

```dockerfile
ENV DOCKER_GROUP_GID <docker_gid>
```

Change user to root for privilege.

```dockerfile
USER root
```

Download docker binary. You can download any version you want.

```dockerfile
RUN wget https://download.docker.com/linux/static/stable/x86_64/docker-18.03.1-ce.tgz
```

Extract docker binary.

```dockerfile
RUN tar -xvf docker-18.03.1-ce.tgz
```

Move extracted docker files into /usr/bin directory.

```dockerfile
RUN mv docker/* /usr/bin/
```

Add group inside docker image which has same gid with host's docker group.

```dockerfile
RUN groupadd -o -g ${DOCKER_GROUP_GID} docker 
```

Put user which runs the jenkins process into the docker group.

```dockerfile
RUN usermod -g docker jenkins
```

Change to non-root user

```dockerfile
USER jenkins 
```

## Build docker image

Build docker image with tag.

```bash
docker build <path_to_dockerfile> -t jenkins
```

## Ship it!

Share the docker socket on host os with the option

```properties
-v /var/run/docker.sock:/var/run/docker.sock
```

The full command will be

```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock -d --name=jenkins -p 8080:8080 -p 50000:50000 --privileged jenkins
```

