---
layout: post
title: JWT Authentication with Salesforce
date: 2021-03-28
categories:
- sysadmin
---

[Salesforce](https://www.saleforce.com), for those of you who might have been living under a rock, is a customer relationship management tool. I work with developers who develop code for salesforce. We follow the usual CI/CD route, but have been dependent on 3rd party tools to authenticate to a Salesforce Org. This is OK, but not ideal, so I set about authenticating to Salesforce with JWT. The official documentation is here: [https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_jwt_flow.htm](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_jwt_flow.htm). This sets out the following steps: 

1) Create an OpenSSL certificate
2) Create an app within the Salesforce Org that you want to authenticate to
3) Upload your SSL certificate to that Org
4) Authenticate with that certificate for the first time (manually)
5) Authenticate using the SSL private key 

## Create an OpenSSL certificate

You will need to have the command line tool `openssl` installed - this can be found (in a *nix based operating system) by typing `which openssl`. If the command line returns something other than an error, then you are set. If you get an error, you will need to install the package `openssl`. 

To create an SSL certificate you should use these following commands:

    mkdir jwt
    cd jwt

Generate a private key and store it in a file called `server.key`:

    openssl genrsa -des3 -passout pass:SomePassword -out server.pass.key 2048
    openssl rsa -passin pass:SomePassword -in server.pass.key -out server.key

Generate a certificate signing request (`.csr` file):

    openssl req -new -key server.key -out server.csr

Generate a self-signed digital certificate from the server request (`.csr` file) and private key (`server.key`):

        openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt

