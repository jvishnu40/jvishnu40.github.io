---
title:      Writing unit test case for AWS infrastructure
layout:     post
category:   Tutorial
tags: 	    [testing, aws, terraform]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
The more you write code regardless of application or infrastructure, the more you desire to bring the best outcome of your writing when they are running as a realtime entity. And no way to deny that testing helps us to gain our confidence before going for production. So, why leave your infrastructure code without testing? In this blog post, I am going to introduce writing unit tests for your infrastructure in AWS.
<!--more-->

Infrastructure as a Code (IAAC) is a blessing to us all, which allows us to code our infrastructure configuration under version control. To reuse a particular piece of code over and over, you have to declare modules just as the same you declare objects in OOP. But what if the base configuration template is not battle-tested or gets prone to configuration drift?  Well, in this case, you better implement unit test cases to cross-check your configuration against acceptance criteria.

# Required tools

* __terraform:__ Terraform is a widely used tool for writing infrastructure code and spinning cloud resources. you can also use existing code configuration by writing modules.
* __kitchen-terraform:__  It is a terraform which is used to test terraform infrastructure configuration
* __rspec:__ Rspec is testing tool for ruby language which is domain specific in nature, easy to write and used to test ruby code
* __awsspec:__ It is rspec test suite for testing AWS infrastructure resources
* __ruby:__ I guess no need to introduce this to you ;) if you can't remember then google :D
* __bundler:__ Manages ruby application dependencies and ruby gems
* __git:__ Do I also need to introduce git, seriously?
* __make:__ Linux build tool to execute task in easy way

# Procedure

* At first make sure that you have git, ruby, bundler, make and terraform installed in your system.

* In the next step, clone [this](https://github.com/shudarshon/terraform-aws-unit-test) git repository. This repository holds tiny Configurations for testing terraform code against AWS. The project structure looks like following ![Project Structure](/assets/img/2019-08-09.png)*Project Structure*.

* From the project structure we see that we got a `main.tf.env` file which is actually a terraform configuration file for a single ec2 instance. But the configuration looks like terraform variable file. You might be thinking why mixing up variables and configurations?

```
module "ec2" {
  source         = "modules/ec2"
  aws_region     = "xxx"
  instance_type  = "xxx"
  instance_name  = "xxx"
  ami_id         = "xxx"
  subnet_id      = "xxx"
  security_group = "xxx"
  ssh_user_name  = "xxx"
  ssh_key_name   = "secret"
  ssh_key_path   = "/home/user/keyfiles/secret.pem"
  instance_count = 1
  dev_host_label = "dev"
}
```

Do you remember earlier I told about reusing of code like OOP? Terraform provides exactly same features through modules. You create a module and later use them for similar type of resources multiple time with their own values. Here terraform module `ec2` is declared under `modules` directory.

# Issue

If you find any issue create an issue [here](https://github.com/shudarshon/terraform-aws-unit-test) or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com
