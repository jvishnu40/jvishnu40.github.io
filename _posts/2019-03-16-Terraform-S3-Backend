---
title:      Using S3 bucket as Terraform Remote Backend
layout:     post
category:   Tutorial
tags: 	    [iaac, terraform, s3]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
If you are familiar with IaC then you must have heard about terraform, right? Terraform is an awesome tool letting you to create, destroy, provision and configure your infrastructure as code. When terraform is instructed to run against some of its configuration then it creates a state file in the developer machine. This state file contains infrastructure configuration as applied by Terraform. But if you want to work on the terraform codebase as a team allowing terraform to create state file locally then evermy member will be in a great dilemma. Its because the other developers would be also requiring access to this file. Well you might think that we can use git, right? But one problem with that is some sensitive fields such as credentials might get exposed and this is not what you want. One soultion is to this criteria is to use terraform S3 backend for storing infrastructure state in S3 bukcet. In this post, we are going to discuss and implement terraform S3 backed in AWS. 
<!--more-->

# Requirements

* AWS CLI Access
* Terraform
* Make (build tool)


# Procedure

* At first, export AWS access key id and secret key to environment variables which will be used by Terraform.

```
$ export AWS_ACCESS_KEY_ID="XXYYYYY"
$ export AWS_SECRET_ACCESS_KEY="ZZZZXXXXX"
```

* Next, clone this repository and get into specific source code repository.

```
$ git clone https://github.com/shudarshon/challenge-terraform.git
$ cd 6-tf-s3-backend
```

# Issue

If you find any issue create an issue [here](https://github.com/shudarshon/challenge-terraform/issues) or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com




