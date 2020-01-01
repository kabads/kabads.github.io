---
title: SELinux Users, Roles, and Types
categories:
- sysadmin
date: 2019-12-31
layout: post
---
SELinux contexts follow the SELinux user:role:type:level syntax.

### Users

Users can be seen with the command: 

    semanage user -l

This will show the list of SELinux users, and is mapped to Linux users. If a Linux user is not listed then they are associated with the ```__default_``` SELinux user and inherit those rights.  
