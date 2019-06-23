---
layout: post
title: "Strings in Ruby"
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

.upcase method does the opposite of downcase. It will change the case of any lowercase characters to uppercase. 

    "My String".upcase
    
returns

    "MY STRING"
 

#### .gsub

.gsub is a way of substituting text in one string for a different string. 

    "Hello".gsub("ell", "orr")
    
returns 

    "Horro"
    
    
### String Interpolation

This is potentially the most useful way of working with strings when outputting them. 

    first_name = "Foo"
    surname = "Bar"
    puts "Hello #{first_name} #{surname}
    
