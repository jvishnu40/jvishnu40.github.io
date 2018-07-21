---
title:      Managing Docker Secret with AWS Parameter Store
layout:     post
category:   Tutorial
tags: 	    [docker, security, aws]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
We write Dockerfile for packaging our application into Docker images. Dockerfile is a set of plaintext instructions with predeclared environment variables. These environment variable contains sensitive information such as username, password, remote database URL, bastion host IP etc. When secret variables will be visible in source code repository then code repository members can see them easily. Obviosuly, we need to tackle this situation! In this post, I will discuss on keeping these values secret in AWS, fetching and using them in docker runtime.
<!--more-->

There is a way of protecting environment variable which is creating separate file with environment variable and pointing the path in Dockerfile. But still it reamins the same. However, you can also use HaschiCorp's Vault to mange Docker secrets which is also very popular. But you need to manage that on your own. Now, lets focus on AWS Parameter Store service and see its advantages:

* Isolates sensitive data from source control
* Enables to track version control of parameter store variables
* Easily integrates with various AWS services

AWS Paramter Store service can be found under System Mangement Tools in EC2 dashboard. In this example, I am not going to write an entire Dockerfile instead I will use a portion in my example.

# Requirements

* AWS CLI Access (note the region you have configured)
* jq
* IAM policies to allow to fetch value from AWS parameter store
* Docker with sudoer user's group membership
* Dockerfile

# Procedure

* Lets assume we have a Dockerfile with following environment variables. We can see that among these 7 variables three variables {SONARQUBE_JDBC_USERNAME, SONARQUBE_JDBC_PASSWORD, SONARQUBE_JDBC_URL} are sensitive. We can also put them blank in Dockerfile and provide the value in runtime of docker container. But that will keep a log in history of the shell.

```
FROM phusion/baseimage:master
MAINTAINER Shudarshon Chaki <sdrsn.chaki@gmail.com>

ENV SONAR_VERSION=7.1 \
    SONARQUBE_HOME=/opt/sonarqube \
    SONARQUBE_JDBC_USERNAME=sonaruser \
    SONARQUBE_JDBC_PASSWORD=sonarpassword \
    SONARQUBE_JDBC_URL=jdbc:postgresql://db.endpoint.com/sonar \
    JAVA_HOME=/usr/lib/jvm/java-8-oracle \
    PATH=$JAVA_HOME/bin:$PATH
```

* First we will create an IAM role with `AmazonEC2RoleforSSM` policy attached. We will bind this role with EC2 instance where Docker will be running.

* Next, create an EC2 instance with above IAM role and install `awscli`. Next, use `aws configure` command and configure only the region where we are going to save secrets in Parameter Store. In this example, EC2 and Parameter store services are of same region.

* Next, put secrets in Parameter store in following way,

```
$ aws ssm put-parameter --name "SONARQUBE_JDBC_USERNAME" --type "String" --value "actualusername"
$ aws ssm put-parameter --name "SONARQUBE_JDBC_PASSWORD" --type "String" --value "actualpassword"
$ aws ssm put-parameter --name "SONARQUBE_JDBC_URL" --type "String" --value "jdbc:postgresql://original.db.endpoint.com/sonar"
```

Now, these parameters are safe. We will use it in the build server which can be ephemeral like a Docker container or any Jenkins slave.

* Now we will create a bash script alongside the Dockerfile for automatically triggering the value change in build pipeline

```
$(aws ssm get-parameters --names "SONARQUBE_JDBC_USERNAME" | jq -r '.Parameters| .[] | "export " + .Name + "=" + .Value + ""')
$(aws ssm get-parameters --names "SONARQUBE_JDBC_PASSWORD" | jq -r '.Parameters| .[] | "export " + .Name + "=" + .Value + ""')
$(aws ssm get-parameters --names "SONARQUBE_JDBC_URL" | jq -r '.Parameters| .[] | "export " + .Name + "=" + .Value + ""')

sed -i "s#SONARQUBE_JDBC_USERNAME=[^ ]*#SONARQUBE_JDBC_USERNAME=${SONARQUBE_JDBC_USERNAME}#" Dockerfile
sed -i "s#SONARQUBE_JDBC_PASSWORD=[^ ]*#SONARQUBE_JDBC_PASSWORD=${SONARQUBE_JDBC_PASSWORD}#" Dockerfile
sed -i "s#SONARQUBE_JDBC_URL=[^ ]*#SONARQUBE_JDBC_URL=${SONARQUBE_JDBC_URL}#" Dockerfile

docker build -t app .

docker run -d --name sonarqube -p 9000:9000 -p 9092:9092 --restart unless-stopped app
```

* Now if you check the Dockerfile it will be containing new values which we have saved in the AWS Parameter Store.

# Issue

If you find any issue or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com
