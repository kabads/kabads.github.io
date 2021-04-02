---
layout: post
title: "Create Docker Container to Test Salesforce CI/CD"
date: 2021-04-03
draft: true
categories:
- sysadmin

Salesforce has a tricky CI/CD path. It allows for sandboxes, for testing. However, developing a CI/CD pipeline is tricky, especially, when you have pipelines that queue up. To get around this problem, you cand create a CI/CD pipeline locally using (https://www.docker.com)[Docker].

If you use Jenkins, Azure Devops, or another remote CI/CD pipeline, you have to wait longer for builds. Usually, the pipeline pulls code and then builds the code. On a shared service this can take a while. You could have this setup locally - the source code could already be available, saving time pulling large repositories, and the compute power is dedicated (some services put you in a queue). You can also write results out locally to a disk, rather than having to navigate to an 'artifact' area of the build. In this case, it would be good to run it in a docker container, deploying to a sandbox (if required). 

Firstly, if you are going to deploy to a Salesforce sandbox, you will need to authenticate via the JWT certificate method (https://kabads.monkeez.org/sysadmin/2021/03/28/salesforce-auth-jwt.html)[detailed here]. This means your docker container can authenticate by itself with no human interaction. 


