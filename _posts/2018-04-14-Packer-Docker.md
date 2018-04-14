---
title:      Building Automated Docker Image in Docker Hub with Jenkins and Packer
layout:     post
category:   Tutorial
tags: 	    [docker, packer, jenkins, orchestration, security, iaac]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
Docker is a tool for building application image and packer is a tool for creating golden image from single source configuration. So docker images can be built from both docker and packer. If you bake them with Jenkins then application image creation and uploading process will be automated. In this article, I have provided an infrastructure-as-code example to demonstrate the idea.

<!--more-->

If you want to build application image using Docker then you must write a Dockerfile. Writing Dockerfile is not very difficult but you have write them carefully for a production environment. You have to keep special concern for minimizing the layers in Dockerfile. Because, more the layers the more option to reverse engineer the image and get the tools and command used in that step. Using `docker history` command we can reverse image the steps of Dockerfile from a docker image. So, if you want to put all your docker configuration steps in a single layer then you can choose packer. So lets try to extract a docker image first,

```shell

$ docker history $IMAGE_ID  
$ docker history $IMAGE_ID --no-trunc
```

In order to build docker image with packer you have packer and docker installed under a user with `sudo` privileges. To write the infrastructure-as-code you can follow the template [here](https://github.com/shudarshon/infrastructure-as-a-code/tree/master/packer/docker). Before you try to build application image you must login to docker hub using `docker login` command once.

```shell

$ git clone https://github.com/shudarshon/infrastructure-as-a-code.git
$ cd infrastructure-as-a-code/packer/docker <br />
$ make validate <br />
$ make build <br />
```

If you want to automate the application packaging process then you can use the command in a Jenkins pipeline. In this repository, under `ansible/app` folder I have kept the build file of JHipster sample web app in order to package a docker image. So If you implement a Jenkins Master-Slave distributed build and replace the artifact in that directory with build pipeline `as a (Jenkinsfile)` then application build, packaging and uploading process to Docker Hub will be totally automated. Since you are also minimizing docker layers to a single step in `docker history` command so it also secure. At last, I will suggest to remove all docker artifacts after packaging process in order to save space in Jenkins slave with the commands

```shell

$ docker system prune -af
```

If you have experience of working with Jenkins or other build automation server then you can find this blog post helpful otherwise it may be difficult to understand. This post is actually written focusing on Jenkins pipeline stages to implement automated application packaging using docker without Dockerfile!
