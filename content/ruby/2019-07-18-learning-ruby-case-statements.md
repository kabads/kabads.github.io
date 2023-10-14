---
categories:
- ruby
date: "2019-07-18T00:00:00Z"
status: publish
title: Case Statements in Ruby - Learning Ruby 3
---
If you are coming from using python, then you will be used to using a lot of ```if...elif...else``` statements to catch conditions in variables. <!--more-->Sometimes this can be tedious - in Ruby it is easier to use ```case``` statements. 

Let's assume that you are checking for ages of people and want to classify them according to their age (it might be a sceranio of runners in a race and you want to find out who won each age range). Here you can use ```when```. 

    group = ""
    age = gets.chomp.to_i
    case age
    when 0..17
      group = "Under 18s"
    when 18..25
      group = "18s to 25s"
    when 26..30
      group = "26s to 30"
    when 30..Float::INFINITY
      group = "Over 30s"
    end
    puts group

This allows us to somewhat simplify the conditions that might come about from different ages. 
