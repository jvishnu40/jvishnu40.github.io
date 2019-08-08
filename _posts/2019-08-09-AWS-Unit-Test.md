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


# Required Tools

* __terraform:__ Terraform is a widely used tool for writing infrastructure code and spinning cloud resources. you can also use existing code configuration by writing modules.
* __kitchen-terraform:__  It is a terraform which is used to test terraform infrastructure configuration
* __rspec:__ Rspec is testing tool for ruby language which is domain specific in nature, easy to write and used to test ruby code
* __awsspec:__ It is rspec test suite for testing AWS infrastructure resources
* __ruby:__ I guess no need to introduce this to you ;) if you can't remember then google :D
* __bundler:__ Manages ruby application dependencies and ruby gems
* __git:__ Do I also need to introduce git, seriously?
* __make:__ Linux build tool to execute task in easy way


# Discussion

* At first make sure that you have git, ruby, bundler and terraform installed in your system.

* In the next step, clone [this](https://github.com/shudarshon/terraform-aws-unit-test) git repository. This repository holds tiny Configurations for testing terraform code against AWS. The project structure looks like following ![Project Structure](/assets/img/2019-08-09.png)*Project Structure*.

* From the project structure we see that we got a `main.tf.env` file which is actually a terraform configuration file for a single ec2 instance. You need to rename that file to `main.tf` and use real values in configuration parameter to make the project work. But the configuration looks like terraform variable file. You might be thinking why mixing up variables and configurations?

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

* Let's look kitchen test configuration file `kitchen.yml`

```
---
driver:
  name: "terraform"
  root_module_directory: "."

provisioner:
  name: "terraform"

platforms:
  - name: "aws"

verifier:
  name: "awspec"

suites:
  - name: "default"
    verifier:
      name: "awspec"
      patterns:
      - "test/unit/ec2/default_test.rb"
```
We need to write the above configuration file because we have installed terraform kitchen testing pluging for testing our infrastructure code against cloud platforms. Configuration file parameter seems to easy to understand. I have put a dot "." as a value of parameter `root_module_directory`. This parameter requires the path of terraform `modules` directory path. In Linux, dot "." means present directory. Since both kitchen configuration file and terraform module root folder is in same directory hence I have used dot "." as a value. Also, `awsspec` is used verify test cases written at `test/unit/ec2/default_test.rb`.

* Now let's see how the project Gemfile is containing the ruby Gems required for the Project

```
# frozen_string_literal: true

ruby '2.6.0'

source 'https://rubygems.org/' do
  gem 'aws-sdk'
  gem 'awspec'
  gem 'kitchen-terraform'
  gem 'kitchen-verifier-awspec'
  gem 'rhcl'
end
gem "test-kitchen"
```

* Now we will discuss on project unit test is written at `test/unit/ec2/default_test.rb`. At first we require the base dependencies that we need to run the test cases. Next, we declare
three variables `config_main, ec2_name & sg_id`. Later two variables hold exact values of  `instance_name` and `security_group` parameter of `main.tf`. After spinning up infrastructure we will compare the instance name of EC2 and security group id of the EC2 against these two variables.

```
# frozen_string_literal: true

require 'awspec'
require 'aws-sdk'
require 'rhcl'

# Parse and load our terraform manifest into config_main
config_main = Rhcl.parse(File.open('main.tf'))
ec2_name = config_main['module']['ec2']['instance_name']
sg_id = config_main['module']['ec2']['security_group']
```

But reading values of `main.tf` is not enough. After spinning up resources we need more parameters to check such as EC2 state, matching security group id etc. And, how do we do that? The answer is simple. We need to grab those values from terraform state file and store them in variable as first part. Then compare those values through our test cases. If you are missing the terraform state file point, then let me clear you that terraform creates a state file after it has finished spinning up a infrastructure for tracking changes. In the below snippet, we load the state file contents into a variable and pater parse specific values into specific folders for comparing EC2 properties.

```
# Load the terraform state file and convert it into a Ruby hash
state_file = './terraform.tfstate.d/kitchen-terraform-default-aws/terraform.tfstate'
tf_state = JSON.parse(File.open(state_file).read)
subnet_id = tf_state['modules'][1]['resources']['aws_instance.DevInstanceAWS']['primary']['attributes']['subnet_id']
root_volume_id = tf_state['modules'][1]['resources']['aws_instance.DevInstanceAWS']['primary']['attributes']['root_block_device.0.volume_id']
```

Finally, now it is time to write the test cases. In the following snippet you will observe that test cases written in gherkin syntax are extremely readable and comparing the newly spinned up EC2 properties against the variables that we declared earlier. Basically, using this test cases we are getting sure that our desired EC2 instance is existing, running, belongs to subnet X, has root EBS volume and has security group Y.

```
# Test the EC2 resource
describe ec2(ec2_name.to_s) do
  it { should exist }
end

describe ec2(ec2_name.to_s) do
  it { should be_running }
end

describe ec2(ec2_name.to_s) do
  it { should belong_to_subnet(subnet_id.to_s) }
end

describe ec2(ec2_name.to_s) do
  it { should have_ebs(root_volume_id.to_s) }
end

describe ec2(ec2_name.to_s) do
  it { should have_security_group(sg_id.to_s) }
end
```


# How to run

* Copy main.tf.env to main.tf file. then use terraform variable values according to module configuration,

 `cp main.tf.env main.tf`

* Install testing dependencies gems by running,

 `bundle install --path vendor/bundle`

* Next, kitchen converge the test scenario which spins up the resources in aws,

 `bundle exec kitchen converge`

* Then verify infrastructure components by running,

 `bundle exec kitchen verify`

* Once testing is done destroy the resources,

 `bundle exec kitchen destroy`


# Issue

If you find any issue create an issue [here](https://github.com/shudarshon/terraform-aws-unit-test) or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com
