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

## Connected App in the Salesforce Org

### Create a Connected App

This connected app is the place where the SSL certificate is held and allows you to connect. Go to Setup, and type 'App' in the quick find box. Then click on 'App Manager' link. 

Click the 'New Connected App' button in the right hand top corner. Give you app a name in the 'Connected App Name' field. You must provide an email address. 

Under the 'API (Enable OAuth Settings)' click the checkbox for 'Enable OAuth Settings'. A dialogue box will appear. You need to add the settings 'Access and manage your data', 'Full Access', 'Perform Requests on your behalf at any time (refresh_token, offline_access)' and 'Provide access to your data via the Web (web)' in the Selected OAuth Scopes. If you need more permissions, add them (or create them in the first place.)

Also click the 'Use Digital Signatures' checkbox. A 'Browse File' button will appear. Here you will need to upload your `server.crt` file that was created in the SSL stage. 

In the 'Callback URL' enter `https://test.salesforce.com/oauth2/token`. If you have a production org that you are connecting to, the `test` part should be replaced with `login`. 

Click the save button. 

You will see, on the newly created app page' a `Consumer Key` - this is often called the `client_id` in the JWT specification. You will need this later on, so make a note of it. 

[URL here for connected app screenshot]

### Manage Profiles and Permission Sets for the Connected App 

The page will load, with your changes saved, but you still need to add information to this app to allow JWT to authenticate. 

Click 'Manage' at the top of the connected app, then 'Edit Policies'. 

In the 'OAuth Policies' section, choose 'Admin approved users are pre-authorized'. A dialogue warning box will appear. Accept the warning. Click save at the bottom of the page. 

Scroll down and click 'Manage Profiles'. Choose a profile that suits the needs of your connection. Then, click on 'Manage Permission Sets' and choose a permission set to use with this profile (or create one if necessary). 

## Authenticate with that certificate for the first time (manually)

Now that you have the app ready to accept connections, you need to authenticate initially 