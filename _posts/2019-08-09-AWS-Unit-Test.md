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

* __terraform:__ terraform is a widely used tool for writing infrastructure code and spinning cloud resources. you can also use existing code configuration by writing modules.
* __kitchen-terraform:__  it is a terraform which is used to test terraform infrastructure configuration
* __rspec:__ rspec is testing tool for ruby language which is domain specific in nature, easy to write and used to test ruby code
* __awsspec:__ it is rspec test suite for testing AWS infrastructure resources
* __ruby:__ i guess no need to introduce this to you ;) if you dont know then google xD
* __bundler:__ manages ruby application dependencies and ruby gems


# Issue

If you find any issue create an issue [here](https://github.com/shudarshon/terraform-aws-unit-test) or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com
