---
title:      Performing Application Build and Remote Deployment with Travis CI
layout:     post
category:   Tutorial
tags: 	    [ci/cd]
#feature:   /assets/img/1_tsunami_udp_data_transfer.svg
---
Travis CI is another popular CI/CD service which is free for open source projects with limited functionality. In the free version, unlike Jenkins, Travis CI builds application in a  dockerized environment whose underlying architecture is can be seen in build log. After a Jenkins setup specially in a Master-Slave model we have to configure a lot of plugins, remote SSHing, spicific user, privileges etc. Instead Travis CI comes with single YAML configuration file where we need specify parameters and configure them to perform successful build and remote deployment. In this topic I will be focusing only on writing Travis CI/CD configuration not signing up Travis CI and integrating it with github source code repository.
<!--more-->

At first, we will create a `.travis.yml` file and `.travis` directory in the root of source code repository. That YAML file is the configuration file of Travis CI and the directory will be used for storing encrypted key, deployment script which will be used by the Travis at build and deployment stage of the application. After creating the file and directory we need to install Travis client in the workstation. After installing Travis client authenticate with GitHub username and password to connect successfully with Travis account.

```sh
sudo gem install travis
travis login
```

While using Travis CI we need to keep in mind that for remote deployment we must SSH from intermediate Travis docker image to remote server. In this case, we need to keep the {public-private key, remote server IP, SSH username and deploy path} encrypted. Travis makes it easy for adding the encryption data directly in the configuration file with a simple `--add` parameter. So lets encrypt the following entities and add them to configuration file,

```sh
cd /path/to/repository
travis encrypt DEPLOY_HOST=X.X.X.X --add
travis encrypt DEPLOY_PATH=/home/user/app --add
travis encrypt DEPLOY_USER=user --add
```
After using the command we will see that following part has been added to Travis configuration file.

```
env:
  global:
  - secure: sDy1io12CHmmSVFDVcrwQgBsz1lIJIBui//6nclVBPOu6cMiNWeYeTsvEyN99VY5z0I2oVhGG30LadhppG6FFFF8SaKJC9Ec/KFTEIxov69Tt5BNtDiwIGjLB5ZLZDi/1Sg16g3GFeSNB79T5MNzLSGGxW5YFVklWPgk1yj7MOYelT62TNFHq2z1C+ZldcetZNvpycyNacK/t+eJ0OFGbFe6MzuCMP2EgaNKxc0YUwIHGmzfU6VRyXAgGrZZQHr4ZBus/GquJhaiDCpjc5cQl3rY7Q+J9SYi8+kyLgzGkphUQWK80NrQ/GaT/J/MQbgGU6WDvAGCBkSlRHyIfN9ME/8qerP4Mu1ASbFf6+OBYQlAHYB1lrtdUCjpdNhLksHbkwCh3hl5CblGlhoT0Kmom2ArUNL5mZwcnZCBFfF8IPPTwEFnzj3d67IJL1/OCqqprjCpHHrKpZ+81pkfVVuYwGaORxsY2rTJVEzsB5QCdOzBgdW937RUnw5X72BfrjlvROcoOvDgFm6ZtnFs+6/CyUlaXMNa587pQtiWKo/uDATONq/Cjrr1J7BTm2KnjxNmFzby3xCarOQ0C+GKd6ptm209/b9Id7UCSlmSsWcowgYKblwDYIE5u8LMtHuI3I6sGQdLYOZShJqdSLYXcQP93BqpEDWs0rbQXSQIdBNHW2s=
```

Now in same directory (project root) implement the following command to generate SSH public-private keypair and encrypt the private key,

```sh
ssh-keygen -t rsa -b 4096 -C 'dev@domain.com' -N '' -f ./deploy_rsa
travis encrypt-file deploy_rsa --add
mv deploy_rsa.enc .travis/
```

So, our private key is encrypted and moved into `.travis` folder but still the public key remains. Travis will decrypt the private key during the deployment stage of the application. But still the public key remains. We need to add that public key in remote server's `authorized_keys` file for SSH approval. (For EC2 instance, better you bake AMI with packer and predefined SSH key )

```sh
cat ./deploy_rsa.pub | ssh -i ~/.key/test.pem DEPLOY_USER@DEPLOY_HOST "cat - >> ~/.ssh/authorized_keys"
rm -f deploy_rsa deploy_rsa.pub
```

After performing the above steps we need to configure SSH agent for getting into the remote macnine with decrypted private key in the Travis configuration file. Add SSH configuration in following way,

```
before_deploy:
- openssl aes-256-cbc -K $encrypted_3f698a17050f_key -iv $encrypted_3f698a17050f_iv -in .travis/deploy_rsa.enc -out /tmp/deploy_rsa -d
- eval "$(ssh-agent -s)"
- chmod 600 /tmp/deploy_rsa
- ssh-add /tmp/deploy_rsa
```

After setting up secrets and SSH configuration now we want to run script in remote server as following,
```
deploy:
  provider: script
  skip_cleanup: true
  script: "/bin/bash .travis/deploy.sh"
  on:
    branch: master
```

The actual deploy script is copying files to remote directory with no `HostKeyCheck` and `KnownHostFile`. Although it allows attacker to connect from Travis docker image to remote server but without this option deployment will not be automated. I tried this option with `addons` by revealing remote server IP and it worked. But that seemed to me more risky. The following script will delete source files after successfully copying files to remote server.

```
rsync -r --delete-after --quiet -e "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null" $TRAVIS_BUILD_DIR/ $DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH
```

Full configuration of Travis CI for this particualr project will be found [here](https://github.com/shudarshon/node-js-sample/blob/master/.travis.yml) and the repository can be found on [github](https://github.com/shudarshon/node-js-sample/).
