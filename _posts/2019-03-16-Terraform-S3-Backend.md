---
title:      Using S3 bucket as Terraform Remote Backend
layout:     post
category:   Tutorial
tags: 	    [iaac, terraform, s3]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
If you are familiar with IaC then you must have heard about terraform, right? Terraform is an awesome tool letting you to create, destroy, provision and configure your infrastructure as code. When terraform is instructed to run against some of its configuration then it creates a state file in the developer machine. This state file contains infrastructure configuration as applied by Terraform. But if you want to work on the terraform codebase as a team allowing terraform to create state file locally then every member will be in a great dilemma. Its because the other developers would be also requiring access to this file. Well you might think that we can use git, right? But one problem with that is some sensitive fields such as credentials might get exposed and this is not what you want. One soultion is to this criteria is to use terraform S3 backend for storing infrastructure state in S3 bukcet. In this post, we are going to discuss and implement terraform S3 backed in AWS.
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

* Since S3 is our choice for using remote backend so create an S3 bucket for storing terraform remote state file.

```
$ aws s3api create-bucket --bucket webapp-terraform-state --region us-east-1
```

* Next, clone this repository and get into specific source code repository vpc resource.

```
$ git clone https://github.com/shudarshon/challenge-terraform.git
$ cd challenge-terraform/6-tf-s3-backend/vpc
```

* From the source code we can see that there are 4 different AWS resources terraform configuration categorized into specific folder by resource name. Lets find out how a terraform remote backend configuration (vpc/backend.tf) looks like

```
provider "aws" {
  region  = "${var.aws_region}"
}

data "aws_availability_zones" "available" {}

################################################################################
## Terraform Remote Backend
################################################################################

terraform {
  backend "s3" {
    bucket = "webapp-terraform-state"
    key    = "vpc/vpc.tfstate"
    region = "us-east-1"
    encrypt = true
  }
}
```

In above configuration we have specified the bucket name along with the S3 key. We need to specify the region of S3 bucket also. Generally, it is a good practice to isolate the resource specific state file into separate folder under S3 bucket.

* Writing the backed configuration is not enough. We need also save the important resource properties in remote state file. For instance, suppose we need to allow VPC CIDR block traffic in a specific security group then we need to use `terraform output` to save that configuration in remote state. Lets take a look how the vpc specific terraform output configuration looks like,

```
output "vpc_id" {
  value = "${aws_vpc.vpc.id}"
}

output "vpc_cidr" {
  value = "${aws_vpc.vpc.cidr_block}"
}

output "public_subnet_id" {
  value = [
    "${aws_subnet.subnet_1_public.*.id}",
    "${aws_subnet.subnet_2_public.*.id}"
  ]
}

output "webapp_subnet_id" {
  value = [
    "${aws_subnet.subnet_1_private_webapp.*.id}",
    "${aws_subnet.subnet_2_private_webapp.*.id}"
  ]
}

output "webapp_subnet_cidr" {
  value = [
    "${aws_subnet.subnet_1_private_webapp.*.cidr_block}",
    "${aws_subnet.subnet_2_private_webapp.*.cidr_block}"
  ]
}

output "rds_subnet_id" {
  value = [
    "${aws_subnet.subnet_1_public_rds.*.id}",
    "${aws_subnet.subnet_2_public_rds.*.id}"
  ]
}

output "cislave_subnet_id" {
  value = [
    "${aws_subnet.subnet_1_private_cislave.*.id}",
    "${aws_subnet.subnet_2_private_cislave.*.id}"
  ]
}

output "spare_subnet_id" {
  value = [
    "${aws_subnet.subnet_1_private_spare.*.id}",
    "${aws_subnet.subnet_2_private_spare.*.id}"
  ]
}

output "rds_subnet_group_name" {
  value = "${aws_db_subnet_group.rds_subnetgroup.*.name}"
}
```

* Now we need to initialize this configuration with terraform and make it create a VPC resource in AWS. After it finishes resource creation it will output the specified resource attributes in the screen also saving them in the S3 bucket's `vpc/vpc.tfstate` file.

```
$ make init
$ make apply
```

* 

# Issue

If you find any issue create an issue [here](https://github.com/shudarshon/challenge-terraform/issues) or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com
