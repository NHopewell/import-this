---
layout: post
title:  "itertools.zip_longest() saves lives"
date:   2021-02-14 10:26:00 -0500
---  
We know how great ```zip()``` is for processing multiple iterators in parrallel. If I have a list of names and another list of the
lengths of those names, I can iterate over both lists at once like so:
```python
>>> names = "nick sam ben larry".split()
>>> name_lengths = [len(name) for name in names]
>>> for name, length in zip(names, name_lengths):
...     print(f"{name} is length {length}")
... 
# nick is length 4
# sam is length 3
# ben is length 3
# larry is length 5
```

One important issue which can come up when using ```zip()``` is that it stops yeilding results once it reaches the length of the smallest list. So if I add another name to the names list, but don't add an accompanying length to the names_lengths list, ```zip()``` will stop once it has exhausted the names_lengths list:
```python
>>> names.append("sarah")
>>> for name, length in zip(names, name_lengths):
...     print(f"{name} is length {length}")
... 
# nick is length 4
# sam is length 3
# ben is length 3
# larry is length 5
```
Where is Sarah? Is she gone? did ```zip()``` kill Sarah?!?!  

This behaviour is fine most of the time because we often want to iterate through one list along with a second list which was derived from the first (such as a list of lengths). But sometimes we want to iterate through both until we exhaust the longest list (not killing poor Sarah).

```itertools.zip_longest()``` comes to the rescue in these situations. It yeilds results until it exhausts the longest list. Here is zip_longest at work with our lists of different lengths:
```python
>>> import itertools
>>> for name, length in itertools.zip_longest(names, name_lengths):
...     print(f"{name} is length {length}")
... 
# nick is length 4
# sam is length 3
# ben is length 3
# larry is length 5
# sarah is length None
```
And there we go, ```itertools.zip_longest()``` saved Sarah!  

Notice above that we probably don't want to print "sarah is length None". Sarah does have a length, Python just doesn't know what it is in this situation. In these cases, we want to provide a value to the optional argument *fillvalue*, which defaults to None.
```python
>>> for n, l in itertools.zip_longest(names, name_lengths, fillvalue="Unknown"):
...     print(f"{n} is length {l}")
... 
# nick is length 4
# sam is length 3
# ben is length 3
# larry is length 5
# sarah is length Unknown
```
```itertools.zip_longest()``` is roughly equivalent to:
```python
def zip_longest(*args, fillvalue=None):
    # zip_longest('ABCD', 'xy', fillvalue='-') --> Ax By C- D-
    iterators = [iter(it) for it in args]
    num_active = len(iterators)
    if not num_active:
        return
    while True:
        values = []
        for i, it in enumerate(iterators):
            try:
                value = next(it)
            except StopIteration:
                num_active -= 1
                if not num_active:
                    return
                iterators[i] = repeat(fillvalue)
                value = fillvalue
            values.append(value)
        yield tuple(values)
```
