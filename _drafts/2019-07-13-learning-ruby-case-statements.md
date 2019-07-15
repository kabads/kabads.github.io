---
layout: post
title: "Case Statements in Ruby - Learning Ruby 3"
date: 2019-07-13
draft: true
categories:
- ruby
---

If you are coming from using python, then you will be used to using a lot of ```if...elif...else``` statements to catch conditions in variables. Sometimes this can be tedious - in Ruby it is easier to use ```case``` statements. 

Let's assume that you are checking for ages of people and want to classify them according to their age (it might be a sceranio of runners in a race and you want to find out who won each age range). Here you can use ```when```. 

    puts "What is your age?"
    age = gets.chomp
    case age
      when age <18
          group = "Under 18s"
      when age >=18 and age < 26
          group = "18s to 25s"
      when age >=26 and age < 30
          group = "26s to 30"
      when age >30
          group "Over 30s"
    end

This allows us to somewhat simplify the conditions that might come about from different ages. 
