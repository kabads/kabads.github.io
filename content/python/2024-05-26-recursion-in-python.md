---
layout: post
title: Recursion in Python
date: 2024-05-26
categories:
- python
draft: true
---

So, I've started the [100 days of code](https://www.100daysofcode.com) again. This is an attempt to keep the brain ticking over and keep coding fun. There are certain things I do at work, that really aren't always that taxing. But they are necessary. 

This is to explore other areas that I typically wouldn't use. I think I haven't used recursion before at work. This is probably unusual for developers, but it's not really in what I do. 

Python has a [recursion limit of 1000](https://docs.python.org/3/library/sys.html#sys.getrecursionlimit) (by default), built-in. This can be overridden. However, this code shows how it works. I've added in the `try: except:` part, because I still like the program to run, even though I know it will cause a problem:

    def recursion():
      print("Hello world")
      try:
        recursion()
      except RecursionError:
        print("Recursion limit reached")
        pass
      # Python has a default recursion limit of 1000
      # Base case is where you want to stop calling the recursive function
    
    recursion()
    
    def recursion_with_limit(count):
      print("Hello world")
      count += 1
      if count < 500:
        recursion_with_limit(count)
      else:
        print("Recursion limit reached")
        pass
    
    count = 0 
    recursion_with_limit(count)

The important thing to remember in this code is the base case, whereby you catch the condition under which you do not run the recursion again.
