---
layout: post
title:  "Lists are not Arrays"
date:   2021-02-12 10:25:00 -0500
---

I often hear new Python devs who come from other languages refer to Python lists as arrays. 
At first glance, this is not a big deal. Lists do behave just like arrays on the surface. They are both sliced in the same way, they are both accessed in the same way, they can both be extended and appended to in the same way, etc. But there are some very key differences between the two.

The first difference is that lists can hold heterogenous data types. This means I can make a list mixed with ints, strings, and nested lists, like this one : 
```python
my_ list = [1, 2, 'nick', [3, 'sam']]
```
This makes lists are more flexible than arrays as arrays are restricted to containing a single data type (usually ints or floats). But, this flexibility results in lists using a lot more memory than arrays.   

The flexibility of lists is so nice that some Python devs do not even know that Python has a C-style array, always opting for a list data structure instead. 

Here is how a double type array is allocated (the first argument denotes the type):
```python
>>> import array
>>> new_array = array.array('d', [1, 2, 3, 4, 5])
>>> print(new_array)
# array('d', [1.0, 2.0, 3.0, 4.0, 5.0])
```

Under the hood, lists and arrays are represented differently in memory. An array pointer points to the head of the array itself (the address of the first element of the array). And the array itself is a contiguous set of elements in memory. So just like an array in C, if we have a pointer to the head, along with the length of the array, we can easily operate on that array.  

Lists, on the other hand, do not contain elements which are contiguously arrange in memory. Instead, a list points to the head of a block of pointers. Each pointer in this block points to a complete Python object somewhere else in memory (each member of the list). 


This is shown below:
![list_pic](../../../images/list_vs_array.png)

By pointing to the head of a pointer block, Python lists give the illusion of being contiguous in memory even though they do not have to be arranged as so. 

**So when should I use a list? When should I use an array?**  

Good question. Because lists do not need to be contiguous in memory nor of the same type, new elements can be appended very efficiently. If you believe your data structure will grow and shrink a lot, it makes sense to use a list. You should also use a list if you need to store many different data types.  

Arrays might be better to use if you need to store a large amount of one data type (typically numeric) and need to do computationally heavy math operations on the data. If you need to do a lot of math, I would recommend using the Numpy array as it is the most optimized data structure for this sort of task. The Numpy array is the data structure core to most Python machine learning libraries (which have to do a lot of linear algebra), and are the building blocks of the Pandas dataframe.

```python
>>> import numpy as np
>>> numpy_array = np.array([1, 2, 3, 4, 5])
>>> print(numpy_array)
# [1 2 3 4 5]
```