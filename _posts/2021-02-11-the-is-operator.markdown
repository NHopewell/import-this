---
layout: post
title:  "The 'is' operator and the interpreter"
date:   2021-02-11 10:25:00 -0500
---
The Python code below evaluates to True given a value of 256, but False when given a value of 257. But why?

```python
>>> x = 256
>>> y = 256
>>> x is y
True

>>> x = 257
>>> y = 257
>>> x is y
False
```

First, let's talk about what ```is``` actually...is. 

Unlike ```==``` which compares *values* for equality, the ```is``` operator checks if both operands refer to the same object.  In other words, ```is``` checks if the identity of both operands match or not. You can check this yourself with ```id()```.

```python
>>> x = 256
>>> id(x)
4381006464
>>> y = 256
>>> id(y)
4381006464
```

We can see that these variables x and y are pointing to the same object in memory (they both have the same ids). This still does not make a lot of sense. If we do the same with a value of 257 given to x and y, we see that their ids are not the same anymore.

```python
>>> x = 257
>>> id(x)
140476436035088
>>> y = 257
>>> id(y)
140476436034992
```
So what's going on here? 

The answer is that 256 is already an object when you launch python, while 257 is not. You can read this in the [documentation](https://docs.python.org/3/c-api/long.html). It states:

>The current implementation keeps an array of integer objects for all integers between -5 and 256, when you create an int in that range you actually just get back a reference to the existing object.

So when we say ```x = 256``` we are giving x a reference to an existing object. If we do the same for y, we get the same reference. But this is not true when we assign an integer to x or y which is outside of this range (-5 to 256).  

So why are only numbers from -5 to 256 stored as objects when you start Python? Simply because these are common numbers which are used a lot. It makes sense to have them already floating around for use.

When we assign 257 to y right after assigning 257 to x, the interpreter does not know that we just created an integer with a value of 257, so it goes ahead and creates another object (with a different id). The same thing happens with immutable objects. Because lists are mutable ```[] is []``` with always return ```False``` but if we do the same with empty tuples (which are immutable), we see that ```() is ()``` will return ```True```.

But it gets even more interesting when we **initialize two variables with the same value on the same line**:

```python
>>> x, y = 257, 257
>>> x is y
True
```

Now, for some reason, ```x is y``` returns ```True``` when both are given a value of 257. Which was not the case when we assigned 257 to these variables on different lines. 

Like previously mentioned, when we set x and y to 257 on different lines, the interpreter doesn't know we already created this object in memory and so a new one is created. But, when we do this all on one line, Python creates a new object AND references the second variable (y) at the same time. Thus giving both x and y a reference to the same object with the same id. Which explains why the is operator acts like it does. 

The explaination for this is one of optimization within an interactive environment (like a repr). When 2 lines are entered into the interpreter they are handled separately, and optimized separately. If you tried to reconstruct this experiment within a ```.py``` file and run it, this behaviour would not happen. It wouldn't happen because the optimization outside of an interactive environment, such as running an entire script, works holistically. 