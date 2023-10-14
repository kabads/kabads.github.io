---
categories:
- ruby
date: "2019-07-29T00:00:00Z"
status: publish
title: Ruby Arrays
---

Ruby has extensive coverage for arrays (sometimes called lists if you are coming from a Python background). <!--more-->Arrays are for collecting information all in one place, or in a list. Arrays are sequential, which means that they don't change order once set. Arrays are mutable (they can change items in the list). 

### Declaring an Array

Here is an example of an array:

    scores = [14, 12, 4, 20, 19]

These scores can be for a particular person in a particular test or activity. 

Items in the list are called like this:

    puts scores[4]

This will return the 5th element from the array, as the first element is numbered 0 (i.e. ```scores[0]```)

### Adding a new element to the array

You may want to add another new item/variable to the array. To do this you append to the array: 

    scores.append(5)

This will add a new score to the array and returns the whole array. 

### Delete from the array

There are at least two main ways that you can delete from an array. The first is to delete a particular item value from the array:

    scores.delete(5)

will remove any item from the array that are equal to 5. The second is:

    scores.delete_at(0) 

which removes from the positional index in the array (0 being the first element). 


