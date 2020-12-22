---
title: Verify Azure Blob Storage Automatically
categories:
- sysadmin
date: 2020-12-22
layout: post
---

### Problem

You want to verify lots of files that have uploaded to Azure Blob Storage? Look no further. https://github.com/kabads/md5sum

So, I had to upload a lot of media assets for work in a hurry as a server was being shut down. We had Azure, and as these were static files, that seemed like a good solution. I think in total, there was roughly 200,000 files. I wanted to md5sum them at each part of the stage. I did that for the huge 5Gb zip file I was given and asked my colleague who provided it to me to do the same. It was good. I could do this on all my machines, but not once the zip file was unzipped. 

### Solution

So, I wrote a script[0]. Doing the local verification was fairly easy. 

Doing the Azure solution was not so easy. Azure store the md5sum in a weird way and lots of people have written about this. Most of my research returned this kind of post. No one seemed to have my huge amount of files problem, but just had the problem whereby the md5sum format wasn't the same. I tried reverse engineering the problem, but found it tricky. Then I hit upon gold-dust:

    import binascii
    ...
    remote_md5 = binascii.hexlify(b)

This turned the md5sum stored in Azure into something that could be compared (and was the same as the typical md5sum command that you would run locally. 

#### How to handle Azure blobs individually with the Python Azure SDK

Get a BlobServiceClinent: 

    blob_service_client = BlobServiceClient.from_connection_string(connection_str)

The connection string is usually picked up from an environment variable that you have set up locally (and is provided nicely in the Azure console). 

Then, get a container client: 

    container = blob_service_client.get_container_client(container=container_name)

Then, with the container, you can list all the blobs: 

    blob_list = container.list_blobs()

Once you have a list of blobs, you can iterate through them and then get a blob client: 

    for blob in blob_list:
    blob_client = blob_service_client.get_blob_client(container=container_name, blob=blob)
    ...

and then then the blob properties and the md5_properties:
    
    a = blob_client.get_blob_properties()
    b = a.content_settings.content_md5
    
Once  I had that I could use my `binascii.hexlify()` magic and everything would be great to write out to a file. 

This file only really solves my problem, but please feel free to run with it and make adaptions. I'm interested in any pull requests that improves it. It does need less 'hard-coding'. 

[0] https://github.com/kabads/md5sum
