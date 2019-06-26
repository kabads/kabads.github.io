---
layout: post
title: "Strings in Ruby - Learning Ruby 2"
date: 2019-06-21
draft: true
categories:
- ruby
---

Ruby has great string support. This post summarises the functions that are available.

This is not designed to be a tutorial for others, but rather a record for myself in learning Ruby. 


#### .downcase 

This method turns any characters in a string to lowercase. 

    a = "My String".downcase 
    
returns
    
    "my string"

#### .upcase

```.upcase``` method does the opposite of downcase. It will change the case of any lowercase characters to uppercase. 

    "My String".upcase
    
returns

    "MY STRING"
 

#### .gsub

```.gsub``` is a way of substituting text in one string for a different string. 

    "Hello".gsub("ell", "orr")
    
returns 

    "Horro"
    
    
### String Interpolation

This is potentially the most useful way of working with strings when outputting them. 

    first_name = "Foo"
    surname = "Bar"
    puts "Hello #{first_name} #{surname}
    
### String Methods 

There are a whole host of methods for strings. You can see them all with: 

   "".methods
   
This will display all the methods that are available for a string. One of the most important ones is ```.chomp```. 

Firstly, we will get a string with the command ```gets``` - this stands for get string. 

    puts "What is your name?" 
    gets = name
    
Once you run this you will get a string with the name that you enter and a ```\n``` on the end - the newline character. This can be annoying, but ```.chomp``` will remove this: 

    gets = name.chomp
   
This means that the string has the newline character removed. 

#### .count

If you wish to count the amount of times a set of characters appear in a string, use the ```.count``` method. 

    "banana".count("a")

will return the number ```3```. 

    "banana".count("ba") 

will return ```4```. 

#### String Operations

    "ban" + "ana" 

will return ```banana```. 


You cannot do the same thing with ```-```.

#### Converting Strings to Integers 


