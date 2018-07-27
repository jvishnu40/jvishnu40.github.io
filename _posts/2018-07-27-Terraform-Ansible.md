---
title:      Infrastructure provisioning and configuration management with AWS, Terraform and Ansible
layout:     post
category:   Tutorial
tags: 	    [iaac, terraform, ansible, cfm, aws]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
Regularly a DevOps has to perform different types of server provisioning, configuration with different architectures for deployment testing, applying and testing IaaC modules or for building dev/stage environment. Terraform is such a tool which helps us to build & manage infrastructure using different cloud vendor API. Ansible is another tool to automate infrastructure configuration changes. In this post, I will discuss on using Terraform and Ansible together to bring infrastructure in desired state quickly.
<!--more-->

To simplify deployment we need to often use golden machine images which will have all of its configuration tested, patched and updated to stable version. To create such machine images we can use these two tools combined. However, our concern first goes to implement provisioning and CFM with instances. From this instances we can create new machine images or we can use Packer to create machine images combined with Ansible. However this post will be limited to Terraform and Ansible only.

# Requirements

* AWS CLI Access
* Terraform
* Ansible
* Git

# Procedure

* At first, export AWS access key id and secret key to environment variables which will be used by Terraform.

```
$ export AWS_ACCESS_KEY_ID="XXYYYYY"
$ export AWS_SECRET_ACCESS_KEY="ZZZZXXXXX"
```

* Next, clone this repository and get into specific source code repository.

```
$ git clone https://github.com/shudarshon/challenge-terraform.git
$ cd 1-terra-ansible
```

* We will collect SSH username, remote host IP address in Ansible host file while Terraform creates the resources. we can do this using using null_resource module in terraform configuration as following,

```
resource "null_resource" "ConfigureAnsibleLabelVariable" {
  provisioner "local-exec" {
    command = "echo [${var.dev_host_label}:vars] > hosts"
  }
  provisioner "local-exec" {
    command = "echo ansible_ssh_user=${var.ssh_user_name} >> hosts"
  }
  provisioner "local-exec" {
    command = "echo ansible_ssh_private_key_file=${var.ssh_key_path} >> hosts"
  }
  provisioner "local-exec" {
    command = "echo [${var.dev_host_label}] >> hosts"
  }
}
```

* While creating instances we need to provision them with essential packages that Ansible needs to connect with the instances. We will gather public ip address of those specific instances which have been provisioned and created successfully.

```
resource "null_resource" "ProvisionRemoteHostsIpToAnsibleHosts" {
  count = "${var.instance_count}"
  connection {
    type = "ssh"
    user = "${var.ssh_user_name}"
    host = "${element(aws_instance.DevInstanceAWS.*.public_ip, count.index)}"
    private_key = "${file("${var.ssh_key_path}")}"
  }
  provisioner "remote-exec" {
    inline = [
      "sudo yum update -y",
      "sudo yum install python-setuptools python-pip -y",
      "sudo pip install httplib2"
    ]
  }
  provisioner "local-exec" {
    command = "echo ${element(aws_instance.DevInstanceAWS.*.public_ip, count.index)} >> hosts"
  }
}
```

* Now we will change host variable name following Terraform variable and run Ansible playbook run command in Terraform configuration as following,

```
resource "null_resource" "ModifyApplyAnsiblePlayBook" {
  provisioner "local-exec" {
    command = "sed -i -e '/hosts:/ s/: .*/: ${var.dev_host_label}/' play.yml"   #change host label in playbook dynamically
  }

  provisioner "local-exec" {
    command = "sleep 10; ansible-playbook -i hosts play.yml"
  }
  depends_on = ["null_resource.ProvisionRemoteHostsIpToAnsibleHosts"]
}
```

* Run the below command to apply Ansible with Terraform,

```
$ terraform init
$ terraform plan
$ terraform apply
```

* After the operation is completed then you can check that `htop` is installed in each of the instances. Alternatively, you can use Ansible role also with your own configuration.

# Issue

If you find any issue create an issue [here](https://github.com/shudarshon/challenge-terraform/issues) or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com
