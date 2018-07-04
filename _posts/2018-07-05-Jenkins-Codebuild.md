---
title:      Integrating Jenkins with AWS CodeBuild
layout:     post
category:   Tutorial
tags: 	    [ci/cd, jenkins, aws]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
Jenkins is a task server which is highly used in performing CI/CD jobs. AWS CodeBuild is a service for performing source code compilation, build, testing and artifact generation for deployment. We can integrate these two two services to scale our build, reduce build time, minimize billing and exploit the combined power of CodeBuild and Jenkins.
<!--more-->

CodeBuild is an excellent tool to perform build and test jobs. But you need to write many AWS Lambda functions with SNS services to notify pre-build and post-build notifications in email or hipchat/slack. On the contrary, Jenkins plugins, shared library makes it easy for you to manage notifications in email, slack and hipchat. But managing jenkins slaves are painful, costly (reserved instances can drastically reduce cost!) and they need to be configured with lots of parameters which makes the scenario complex. As a DevOps you need to achieve several CI/CD goals for faster software release in a cost effective way. Sometimes you will be facing the dilemma of scaling build vs reducing infrastructure cost. Thats exactly where we will be using Jenkins with CodeBuild. Jenkins will use CodeBuild infrastructure to scale, perform build tasks on source code change with proper notifications of pre and post build results.

If you find any issue or want more feature then create an issue [here](https://github.com/shudarshon/challenge-jenkins/tree/master/7-jenkins-codebuild).
