---
title:      Automated Tomcat Installation with Ansible
layout:     post
category:   Tutorial
tags: 	    [tomcat, ansible, cfm]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
In this post, I will discuss on setting up tomcat application server in automated way using ansible. Ansible makes it easier for us to remotely manage configuration of different servers.

<!--more-->

# Prerequisite

You just need one linux server with SSH access from local machine. I will be using Ubuntu 16.04.3 in AWS environment. Make sure you have 8080 port open in the firewall. Also you need to install python in remote server otherwise ansible will not able to run remote tasks there,

```shell
ubuntu@remote-server:~$ sudo apt-get update
ubuntu@remote-server:~$ sudo apt-get install python-setuptools -y
```

# Install ansible and configure

First, you need to install ansible in the local machine. Use following commands,

```shell
ubuntu@local-machine:~$ sudo apt-add-repository ppa:ansible/ansible -y
ubuntu@local-machine:~$ sudo apt-get update
ubuntu@local-machine:~$ sudo apt-get install ansible -y
```

Now change the `host_key_checking` parameter of ansible configuration file to `False`. Open `/etc/ansible/ansible.cfg` file with vim and change the parameter value. Then save and exit.

# Configure ansible role setting

You will need java to run tomcat in remote server. So you need to install both java and tomcat there. Following commands will help you to setup java and tomcat in remote server

```shell
ubuntu@local-machine:~$ git clone https://github.com/shudarshon/ansible_role.git
ubuntu@local-machine:~$ cd ansible_role
ubuntu@local-machine:~/ansible_role$ vim hosts   # change the target serer ip and set username with private key file location accordingly. then save and quit
ubuntu@local-machine:~/ansible_role$ ansible -i hosts play.yml #this will start setting up tomcat and java in the server
```

# Install

I assume that already you have completed setting up ansible and configured `hosts` file with remote server information. Now, run ansible to perform java and tomcat setup in the target server
```shell
ubuntu@local-machine:~/ansible_role$ ansible -i hosts play.yml
```

Now, browse to http://SERVER_IP:8080. For ease of task, I have saved tomcat admin/manager username/password as `admin/adminsecret` as plaintext in the ansible role. Its not a good practice at all. Later, on another post I will try to implement secure credential management in ansible.
