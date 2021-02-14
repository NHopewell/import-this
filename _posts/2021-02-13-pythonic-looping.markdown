---
layout: post
title:  "Pythonic looping"
date:   2021-02-13 10:25:00 -0500
---  

You've probably realized that Pythons most common for loop is actually a foreach loop. A foreach loop does not keep track of any counter. It says "do something for each item in this collection" rather than "do something x number of times" like a traditional for loop does. If you have an iterable data structure that you want to iterate through, you can loop over its items directly like so:

```python
>>> names = 'nick sam ben larry'.split()
>>> for name in names:
...     print(name)
... 
# nick
# sam
# ben
# larry
```

```range()``` is a builtin which is nice for looping over a sequence of integers easily:
```python
>>> for i in range(1, 11, 2):
...     print(i)
... 
# 1
# 3
# 5
# 7
# 9
```

But its quite common to want to loop over all the items in a list while also keeping track of the index of each item. A non-pythonic
way to do this is to use range (which is meant for looping over integers) like so:
```python
>>> for i in range(len(names)):
...     print(f"{i}: {names[i]}")
... 
# 0: nick
# 1: sam
# 2: ben
# 3: larry
```

This is needlessly clunky. We need to compute the length of the array and index into the array. As a rule of thumb, Pythonic syntax is beautiful and expressive. If the code looks clunky, it's usually not the Pythonic way of doing things. Python does have a builtin specifically designed for this task: ```enumerate()```.

```enumerate()``` **will wrap any iterator with a lazy generator** which will yield tuples each time next is called. The tuples that enumerate yields are the index of the item and the item itself. Like this: (index, item). We can keep calling next on our names list directly to see this in action:

```python
>>> names_enum = enumerate(names)
>>> print(next(names_enum))
# (0, 'nick')
>>> print(next(names_enum))
# (1, 'sam')
>>> print(next(names_enum))
# (2, 'ben')
>>> print(next(names_enum))
# (3, 'larry')
```

We can use a for loop to call next until stop iteration occurs, each loop yeilding these tuples, like so:
```python
>>> for i, name in enumerate(names):
...     print(f"{i}: {name}")
... 
# 0: nick
# 1: sam
# 2: ben
# 3: larry
```

And something that many Python developers forget is that ```enumerate()``` **takes a second argument** which is the index to start with:
```python
>>> for i, name in enumerate(names, 1):
...     print(f"{i}: {name}")
... 
# 1: nick
# 2: sam
# 3: ben
# 4: larry
```
