---
layout: post
title: JWT Authentication with Salesforce
date: 2021-03-28
categories:
- sysadmin
---

{{< rawhtml >}}
<img src='https://jwt.io/img/pic_logo.svg' alt='JWT Logo'>
{{< /rawhtml >}}

[Salesforce](https://www.saleforce.com), for those of you who might have been living under a rock, is a customer relationship management tool. I work with developers who develop code for salesforce. <!--more-->We follow the usual CI/CD route, but have been dependent on 3rd party tools to authenticate to a Salesforce Org. This is OK, but not ideal, so I set about authenticating to Salesforce with JWT. The official documentation is here: [https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_jwt_flow.htm](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_jwt_flow.htm). This sets out the following steps: 



1) Create an OpenSSL certificate
 
2) Create an app within the Salesforce Org that you will authenticate to

3) Manage Permissions for the Managed App

4) Authenticate using the SSL private key 

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

<a data-flickr-embed="true" href="https://www.flickr.com/photos/kabads/51010452920/in/dateposted/" title="connected-app-01"><img src="https://live.staticflickr.com/65535/51010452920_64c13d64d8.jpg" width="500" height="240" alt="connected-app-01"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

### Manage Profiles and Permission Sets for the Connected App 

The page will load, with your changes saved, but you still need to add information to this app to allow JWT to authenticate. 

Click 'Manage' at the top of the connected app, then 'Edit Policies'. 

In the 'OAuth Policies' section, choose 'Admin approved users are pre-authorized'. A dialogue warning box will appear. Accept the warning. Click save at the bottom of the page. 

<a data-flickr-embed="true" href="https://www.flickr.com/photos/kabads/51079734032/in/photostream/" title="connected-app-02PNG"><img src="https://live.staticflickr.com/65535/51079734032_2d160ef78f.jpg" width="500" height="127" alt="connected-app-02PNG"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

Scroll down and click 'Manage Profiles'. Choose a profile that suits the needs of your connection. Then, click on 'Manage Permission Sets' and choose a permission set to use with this profile (or create one if necessary). 

## Authenticate using the SSL private key 

Now that you have the app ready to accept connections, you need to authenticate initially using a browser, or an API tool like Postman or Advanced Rest Client. For simplicity, we are going to use the browser. 

Then, type in the command line 

    sfdx auth:jwt:grant --clientid=3MVG9SHET737DDSDB4lkkcuR.z3NkG98GEIq5h9hcF.YBNJ.PkDOEDE66785AODEDEa78TvyzcJ \ 
    --jwtkeyfile=./server.key \
    -u your-user@orgname-dedod.com

You will then be authenticated to your organisation and will be able to type in commands. This method allows for CI/CD pipelines to authenticate themselves. 
