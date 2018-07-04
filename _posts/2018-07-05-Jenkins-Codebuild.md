---
title:      Integrating Jenkins Pipeline with AWS CodeBuild
layout:     post
category:   Tutorial
tags: 	    [ci/cd, jenkins, aws]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
Jenkins is a task server which is highly used in performing CI/CD jobs. AWS CodeBuild is a service for performing source code compilation, build, testing and artifact generation for deployment. We can integrate these two services to scale our build, reduce build time, minimize billing and exploit the combined power of CodeBuild and Jenkins.
<!--more-->

CodeBuild is an excellent tool to perform build and test jobs. But you need to write many AWS Lambda functions with SNS services to notify pre-build and post-build notifications for email or hipchat/slack notifications. On the contrary, Jenkins plugins, shared library makes it easy for you to manage notifications for email, slack and hipchat. But managing jenkins slaves are painful, costly (reserved instances can drastically reduce cost!) and they need to be configured with lots of parameters which makes the scenario complex. As a DevOps you need to achieve several CI/CD goals for faster software release in a cost effective way. Sometimes you will be facing the dilemma of scaling build vs reducing infrastructure cost. Thats exactly where we will be using Jenkins with CodeBuild. Jenkins will use CodeBuild infrastructure to scale, perform build tasks on source code change with proper notifications of pre and post build results.

# Requirements

* AWS Account ID
* AWS CLI Access (with Full Access privilege)
* Jenkins Server with CodeBuild plugins installed
* Source Code Repository (GitHub/BitBucket/CodeCommit)
* CodeBuild Region information

# Procedure

* First, we need to create IAM group and user with appropriate policies from AWS console. IAM Group policies might include `AmazonS3FullAccess (optional), AWSCodeBuildDeveloperAccess`. User should be assigned in the specified group and should be given programmatic access to AWS resources. After creating the user we need to generate AWS access key id and secret access key for using them in jenkins credential.

* Next, we need to create IAM role which will allow CodeBuild to assume policy specified user privileges to use AWS resources. Make sure you have changed CodeBuild project name, IAM username, region in the policy file. Following command will create IAM role,

```
$ cd iam-codebuild-role-policy/
$ aws iam create-role --role-name CodeBuildServiceRole --assume-role-policy-document file://create-role.json
$ aws iam put-role-policy --role-name CodeBuildServiceRole --policy-name CodeBuildServiceRolePolicy --policy-document file://put-role-policy.json
```

If you forget AWS Account ID then retrieve it by using following command,

```
$ aws sts get-caller-identity --output text --query 'Account'
```

* After creating the IAM role we need to create Jenkins credential with IAM user AWS access credential. While creating credential in Jenkins we must choose credential type as `CodeBuild Credentials type` with appropriate IAM user access key ID and secret key. Then click on `Advanced` button and provide the ARN of IAM role in the specific field. Then save the configuration and make sure that no error appears in the Jenkins credentials configuration page. Gather the credential ID for later use in Jenkinsfile.

* Configure CodeBuild from AWS console with SCM source, artifact S3 bucket(optional), specific IAM role and put `buildspec.yml` in project root directory along with Jenkinsfile.

* Make sure you use proper value of variables in Jenkinsfile,

```
#!/usr/bin/env groovy

import jenkins.model.*

node {
  stage('Build') {
    awsCodeBuild projectName: "$CODEBUILD_PROJECT_NAME", region: "$REGION", sourceControlType: "project", credentialsId: "$JENKINS_CREDENTIAL_ID", credentialsType: "jenkins"
  }
}
```
Also we need to allow script approval in Jenkins otherwise it will throw error during build process.

* Optionally, you can add shared library in the Jenkinsfile for pre and post build notifications.

* If your project uses remote database (RDS) then you need to allow CodeBuild IP address in firewall rule. Gather CodeBuild service IP range from https://ip-ranges.amazonaws.com/ip-ranges.json.

* If everything is fine then trigger a build from Jenkins and you will get build log right in the Jenkins console log. Hurray, so no more window switching to CodeBuild for viewing build logs!

# Issue

If you find any issue or want more feature then create an issue [here](https://github.com/shudarshon/challenge-jenkins/tree/master/7-jenkins-codebuild).
