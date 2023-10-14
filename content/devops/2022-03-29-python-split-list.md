---
layout: post
title: Python - split a list in to smaller lists
date: 2022-03-29
categories:
- devops
---

So, you end up with a big list, that has a lot of entries, and you want to split them in to smaller lists. 
<!--more-->
My niece called me recently. She is doing her MA in Neuropsychology and has a set of readings from her research, where she wants to get a median for every minute. That means, she wants to split all her readings into smaller list, containing 60 entries. 

Assuming that I've already split all her ~800,000 entries into a big list with:

    import csv
    import statistics 

    datalist = []
    with open(filename, 'r', encoding='utf-8-sig') as csvfile:
        datareader = csv.reader(csvfile)
	for row in datareader:
            datalist.append(float(row[0]))

    MAX_SIZE = 60

Then, I found via google this one liner, which is great.

    composite_list = [datalist[x:x+MAX_SIZE] for x in range(0, len(datalist),MAX_SIZE)]

This is a great little one liner. It is a [List Comprehension](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions), that creates a list using a slice. 

It will return a list of lists, which is my situation, is great. 
