---
title:      Securing Wordpress Application Secret and Managing Jenkins Pipeline with EC2 Parameter Store
layout:     post
category:   Tutorial
tags: 	    [jenkins, security, aws]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
After installing Wordpress when you setup application it will create a file named `wp-config.php` in the root of the source code. This file contains sensitive informations like database url, username, password and database name. We need mask those values with fake one and replace them with original one before build process. In this post, I will discuss on keeping these values secret with EC2 parameter store and using them in Jenkins pipeline.
<!--more-->

However, you can also use HaschiCorp's Vault to mange secrets which is also very popular. But you need to manage that on your own. Now, lets focus on AWS Parameter Store service and see its advantages:

* Isolates sensitive data from source control
* Enables to track version control of parameter store variables
* Easily integrates with various AWS services

AWS Parameter Store service can be found under System Management Tools in EC2 dashboard. In this example, I am not going to write an entire Jenkins pipeline instead I will use a portion in my example.

# Requirements

* AWS CLI Access (note the region you have configured)
* IAM policies to allow to fetch value from AWS parameter store


# Procedure

* First we will create an IAM role with `AmazonEC2RoleforSSM` policy attached. We will bind this role with EC2 instance where Jenkins slave will be running.

* Next, create an EC2 instance with above IAM role and install `awscli`. Next, use `aws configure` command and configure only the region where we are going to save secrets in Parameter Store. In this example, EC2 and Parameter store services are of same region.

* Now let's discuss on the following snippet. This contains four sensitive informations such as `DB_NAME`, `DB_USER`, `DB_PASSWORD` & `DB_HOST`. We need to replace the real values with the fake values and then make a git commit.

```
// ** MySQL settings ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'fakedbname' );

/** MySQL database username */
define( 'DB_USER', 'fakeusername' );

/** MySQL database password */
define( 'DB_PASSWORD', 'fakepassword' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

```

* Now, put secrets in Parameter store in following way,

```
$ aws ssm put-parameter --name "DATABASE_NAME" --type "String" --value "realdatabasename"
$ aws ssm put-parameter --name "DB_USERNAME" --type "String" --value "realusername"
$ aws ssm put-parameter --name "DB_PASSWORD" --type "String" --value "realpassword"
$ aws ssm put-parameter --name "DB_URL" --type "String" --value "original.db.endpoint.com"
```

Now, these parameters are saved as plaintext in AWS. For extra layer of security you can also use AWS KMS to encrypt these values.

* Lets write a Jenkins pipeline script which can fetch the secrets and replace `wp-config.php` file with original values just before the build happens. Jenkins will run on seprate slave instance so there is less chance that any unauthorized user can this process. We will store the secrets as Jenkins environment variables.

```
pipeline {  
  agent any  
  environment {
        //tr command is used with a pipe to remove double quote at the first and last character of the output
        //trailing slash is used to skip single quote in tr command

        DB_USERNAME = '$(/var/lib/jenkins/.local/bin/aws ssm get-parameters --region us-east-1 --names DB_USERNAME --query Parameters[0].Value | tr -d \'"\')'
        DATABASE_NAME = '$(/var/lib/jenkins/.local/bin/aws ssm get-parameters --region us-east-1 --names DATABASE_NAME --query Parameters[0].Value | tr -d \'"\')'
        DB_PASSWORD = '$(/var/lib/jenkins/.local/bin/aws ssm get-parameters --region us-east-1 --names DB_PASSWORD --query Parameters[0].Value | tr -d \'"\')'
        DB_URL = '$(/var/lib/jenkins/.local/bin/aws ssm get-parameters --region us-east-1 --names DB_URL --query Parameters[0].Value | tr -d \'"\')'
  }
  stages {    
    stage('Prepare') {
      steps {
        //trailing slash is used to escape double quotes within double quotes in sed commands

        sh 'cd /var/lib/jenkins/workspace/${PROJECT_NAME}/'
      	sh "sed -i \"/DB_NAME/s/'[^']*'/'${DATABASE_NAME}'/2\" wp-config.php"
      	sh "sed -i \"/DB_USER/s/'[^']*'/'${DB_USERNAME}'/2\" wp-config.php"
      	sh "sed -i \"/DB_PASSWORD/s/'[^']*'/'${DB_PASSWORD}'/2\" wp-config.php"
      	sh "sed -i \"/DB_HOST/s/'[^']*'/'${DB_URL}'/2\" wp-config.php"
      }
    }
    stage('Build'){
      //
    }
    stage('Test'){
      //
    }
    stage('Backup'){
      //
    }
    stage('Deploy'){
      //
    }    
  }    
}
```

* I have used absolute path of aws program above. You should change the according to your configuration. Or you can use $(which aws) inside

* Next, keep this `Jenkinsfile` in the root of the source code and perform git commit followed by a Jenkins build. Make sure the delete the workspace after the build has been finished. You will find new changes in deployed application. If any problem occurs then you will face database test connection error.

* If you want to mange Dockerfile secret or integrate Jenkins-Docker with EC2 parameter store then you can visit my previous post from [here](http://shudarshon.com/2018-07-21/Docker-Secret-Paramter-Store.html).

# Issue

If you find any issue or want more feature/new tutorials then mail me at sdrsn.chaki@gmail.com
