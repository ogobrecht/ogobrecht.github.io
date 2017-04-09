---
title: "Cheat Sheet: Docker"
subtitle: My personal notes around Docker
tags: [cheatsheet, docker]
lang: en
---

- Install [Docker](https://www.docker.com)
- Commandline help: `docker --help`


## Images

- List images: `docker images`
- List all images: `docker images -a`
- Remove image: `docker rmi`


## Container

- List all running containers: `docker ps` or `docker container ls`
- List all containers: `docker container ls -a`
- List all processes of a container: `docker top <container>`
- Remove one or more container: `docker rm <container>`
- Remove all stopped containers: `docker container prune`


## Change port mapping of existing container

- [Stackoverflow](http://stackoverflow.com/questions/19335444/how-do-i-assign-a-port-mapping-to-an-existing-docker-container)


docker run \
  --name orclapex51 \
  --shm-size=1g \
  -p 1551:1521 \
  -p 8051:8080 \
  oracle/database:11.2.0.2-xe

  docker run \
    --name orclapex42 \
    --shm-size=1g \
    -p 1542:1521 \
    -p 8042:8080 \
    oracle/database:11.2.0.2-xe
