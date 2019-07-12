---
layout: post
date: 2019-06-16
categories: ruby
title: Learning Ruby - First Impressions 
---
Chef is used in my place of work, and so as a result I started learning [Ruby programming language](https://www.ruby-lang.org). <img src="https://live.staticflickr.com/65535/48072657641_7545908162_m.jpg" width="180" height="156" alt="ruby-icon">

I know some python (enough to get me by) but thought that knowing some Ruby would be a good development in my learning. This is an attempt an my first impressions of what is good for a beginner and what is not so good (from the perspective of someone coming from Python).

#### Good Points

Ruby is super easy to pick up and get going. Declaring methods and passing variables just works. Casting from one type of variabl to another just seems to happen seemlessly. Some beginners have this problem in Python, but Ruby removes all this worry from the developer and it *just works* (I'm expecting Pythonistas to hate on me here, as it does work flawlessly in Python, but feels more complex to me). Also, the [.each](https://ruby-doc.org/core-2.6.3/Enumerable.html) method is really nice. Being able to do this, just feels good:

    array.each do |item|
       item.do_some_stuff
    end

This makes sense and does read really well.

I really like the string interpolation - again, from a beginner's perspective, this just works out of the box and makes sense:

    name = "Alfie Spoons"
    puts "Hello #{name}."

The way this works so intuitively, helped me realise that this was designed with the developer in mind. 

I usually try to write a few simple programs before I make a judgement - these usually include a command line calcultor (which takes two numbers and an operator and then checks them and goes through the obligatory operator check to work out which function/method needs to receive the two numbers). Another one is a 'guess the number' game, where a random number is created andthen the user has to try and guess it with the fewest guesses (guesses being counted), whilst the only feedback is "You guessed too high/low". For this I really liked the concept of:

    win = false 
    until win do
        ....
    end

Again, this feels intuitive. I know this can be done with a while, but the command until kind of makes sense.

#### Quirks

Not everything is that great - there is always a downside to a language. In this case it's blocks. Now, I'm not sure if I understand code blocks at the moment, when compared to Python functions. I think they are similar to functions, and they work with the command yield (so they can be passed around), but they are not quite as intuitive as passing around a function in Python (where it really is that easy to pass a function in to code as though it's a variable.

Also, when working with a virtual environment (the equivalent of venv in Python), it doesn't feel like you are in a new environment, when Python makes it clear that you have new environment variables. This is what made me give up on the blogging platform [Jekyll](https://www.jekyllrb.com) as installing it became a pain with its dependencies that would mess up other applications that I used. However, since [bundler](https://bundler.io) has been developed and introduced, it's easier now to run different commands with a bundle of depenedencies. 

