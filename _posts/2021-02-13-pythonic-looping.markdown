---
layout: post
title:  "Yielding tuples: pythonic looping"
date:   2021-02-13 10:25:00 -0500
---  

You've probably realized that Pythons most common for loop is actually a foreach loop. A foreach loop does not keep track of any counter. It says "do something for each item in this collection" rather than "do something x number of times" like a traditional for loop does. If you have an iterable data structure that you want to iterate through, you can loop over its items directly like so:

{% gist 560646bba2fb8823345dc5dc33424af7 %}

```range()``` is a builtin which is nice for looping over a sequence of integers easily:
{% gist 6fa793afa6c64ab290fe403f4721d4e4 %}

But its quite common to want to loop over all the items in a list while also keeping track of the index of each item. A non-pythonic
way to do this is to use range (which is meant for looping over integers) like so:
{% gist 386a28bf781694c6740eb3ca3961e505 %}

This is needlessly clunky. We need to compute the length of the array and index into the array. As a rule of thumb, Pythonic syntax is beautiful and expressive. If the code looks clunky, it's usually not the Pythonic way of doing things. Python does have a builtin specifically designed for this task: ```enumerate()```.

```enumerate()``` **will wrap any iterator with a lazy generator** which will yield tuples each time next is called. The tuples that enumerate yields are the index of the item and the item itself. Like this: (index, item). We can keep calling next on our names list directly to see this in action:

{% gist 5b57ab2714a1b52f3610951eb7079b16 %}

We can use a for loop to call next until stop iteration occurs, each loop yeilding these tuples, like so:
{% gist 00ac37af9f776830ee400722f854581c %}

And something that many Python developers forget is that ```enumerate()``` **takes a second argument** which is the index to start with:
{% gist 217b6991dada6fb3cda7f3c8a45339be %}