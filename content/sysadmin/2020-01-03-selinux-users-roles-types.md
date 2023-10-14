---
categories:
- sysadmin
date: "2020-01-03T00:00:00Z"
title: SELinux Users, Roles, and Types
markup: mmark
---
SELinux contexts follow the SELinux user:role:type:level syntax.<!--more-->

### Users

Users can be seen with the command: 

    semanage user -l

This will show the list of SELinux users, and is mapped to Linux users. If a Linux user is not listed then they are associated with the ```__default__``` SELinux user and inherit those rights.  

### Roles

Role based access control is a go-between from the user to the domain. By assigning roles, a user can access particular domains. This determines which domains can be accessed by the SELinux user. The role dictates what domains (types, or contexts) it is possible to be in. 

To find out which domains (roles) a user is approved for: 

    seinfo -ruser_r -x 

Users can switch roles if they want. However, they can only do this if their SELinux is allowed to be in that role. You can check if this is allowed with: 

    semanage user -l

This will list a user and the roles that they are allowed to switch to. Obviously for this to work, the Linux user has to have the corresponding SELinux username when they issue ```id -Z```. 

### Type 

The type is an attribute of Type Enforcement. The type defines a domain for processes, and a type for files. SELinux policy rules define how types can access each other, whether it be a domain accessing type, or a domain accessing another domain. Usually types are defined by the suffix ```_t```. Rather than have many many rules which identify the access between actor and object (e.g. user and file), SELinux has a label attached to each actor and object, which specifies whether the actor is allowed access to the object. The rule specifies if the actor has a label that allows access to the objects (with that label). 

### Domains 

In SELinux labels assigned to a process is also called a ```domain```. An example of an SELinux domain is ```system_u:system_r:named_t```, although that is often reduced to just the type-part, i.e. ```named_t```. 

### Conclusion

This table represents the structure of a label:

```system_u:object_r:lib_t``` associates with an actor ```user_u:user_r:user_t``` in the following way:

{.table}
| ```system_u``` | ```object_r```| ```lib_t``` |
|-----------------|-----------------|--------------
| SELinux User | SELinux Role | SELinux Type
|----------------|---------------|-------------
| ```user_u``` | ```user_r``` | ```user_t``` 
