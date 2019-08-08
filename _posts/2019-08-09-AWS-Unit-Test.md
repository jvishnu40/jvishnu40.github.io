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


# Issue

If you find any issue create an issue [here](https://github.com/shudarshon/terraform-aws-unit-test) or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com
