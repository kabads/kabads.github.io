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

You can choose any operating system based machine, a good candidate is some Linux flavour. In this case, we will be using Ubuntu 18.04 LTS. This gives a good range of tools. We buid the image locally with this `Dockerfile`:

    FROM ubuntu:18.04
    ENV TZ=Europe/London
    RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
    RUN apt-get update -y 
    RUN apt-get upgrade -y
    RUN apt-get install jq -y
    RUN apt-get install wget -y 
    RUN apt-get install xz-utils -y 
    RUN apt-get install vim -y 
    RUN apt-get install ed -y
    RUN apt-get install python -y 
    RUN wget https://developer.salesforce.com/media/salesforce-cli/sfdx-cli/channels/stable/sfdx-cli-linux-x64.tar.xz -O /opt/sfdx-cli-linux-x64.tar.xz
    RUN useradd -d /opt/sfdx salesforce
    RUN mkdir -p /opt/sfdx/sfdx
    RUN tar xJf /opt/sfdx-cli-linux-x64.tar.xz -C /opt/sfdx/sfdx --strip-components 1
    RUN chmod +x /opt/sfdx/sfdx/* 
    RUN /opt/sfdx/sfdx/install
    
This creates an Ubuntu container, with the required tools, most importantly, the `sfdx` cli tool. 

Build this image with: 

    docker build . -t salesforce-pipeline

    services:
        sfdx:
            image: "salesforce-pipeline"
            volumes:
                - ./certs:/opt/cert
                - ./build-code:/opt/build-code
            entrypoint: "/opt/build-code/print.sh 'hello world'"
            environment: 
                username: ${username}
                clientid: ${clientid} # This is not used in the build-code but is used to authenticate sfdx 

This holds two important variables. 
