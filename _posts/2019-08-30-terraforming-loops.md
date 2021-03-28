---
layout: post
date: 2019-08-30
title: Terraform and Loops - creating multiple resources
categories:
- sysadmin
draft: true
---
Terraform is a great tool for creating infrastructure. It's almost becoming the 'de-facto' standard. This post will outline how to create multiple resources with just one code block. 

The oldest form of looping through a Terraform code block is the ```count``` primitive. By defining this, the run of terraform will loop over that resource for the count that has been defined. However, when defining names, the count will need to be used to name the resoure, as follows:

    resource "aws_instance" "instance" { 
      count  = 3 
      name   = "web-${count.index}"
      ami    = "ami-1234567" 
    }

This will create three ec2 instances, with names of ```web-0```, ```web-1``` and ```web-3```. 

Once you use count on a resource, it becomes a list of resources. So in this case ```aws_instance.instance``` is now a list of ec2 instances. If you want to refer to one of the instances then you must use the notation ```aws_instance.instance[0]```.

If you wish to refer to all resources in a list, then you will need to use the splat, as in: 
    ```aws_instance.instance[*].id``` 

Which will return all the ids of the instances created above. 

Limitations
-------------

If you remove items from a list, then the list will rename items. Imagine having instances and then removing the id to one of them. In a list of instances, if you delete the first one (element [0]) from the list variable, instead of deleting that resource, it will remain, but renamed under the second item in the list (element [1]).
