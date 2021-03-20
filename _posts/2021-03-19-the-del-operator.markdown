---
layout: post
title:  "del-ete me: How Python deletes objects"
date:   2021-03-19 10:26:00 -0500
---
Let's talk about deleting objects in Python, object reference counts, and the behaviour of ```__del__```. First, let's create a class which prints out a message when ```__del__``` is called. And immediately after we will create a few objects and delete them. Notice when the message gets printed out. 

{% gist bbb0a9f0f68b64f69eba97a48314d1b5 %} 

At first glance, you might think ```__del__``` is called whenever ```del``` is used on a Python object. But we can see in the code snippet above that this is not the case. We called ```del``` twice and yet it seems our new class called ```__del__``` only once, printing out our message when we deleted the second object y on line 10. But our message was not printed when we deleted x. 

When I mentioned we will be learning a bit about object reference counts, you may already have a good idea of why this behaviour is happening. But let's first recreate this experiment above, with an added twist to maybe add some confusion. 

We will delete x and y again, but this time just before deleting y, we will reference it by checking its type and location in memory. Then, after deleting y, we will check for its presence in the global namespace to confirm it got deleted. Again, pay close attention to when our message gets printed out. And compare where this message gets printed out to our first example. 

{% gist 411ccf86ea91367853b95f62e6d3a36f %}

Once again, our message is not printed out when we delete x. But this time, the message also does not get printed out when we delte y. In our first example it did get printed out when we deleted y. The key to this situation will be explained below, but it has to do with the fact that I read from y in line 6 before deleting on line 9. Something I did not do in the first example. 

So what is going on here?

It is clear that ```del x``` does not simply call ```x.__del__()```. There is some other magic happening behind the scenes. 
When ```del x``` is ecountered, Python deletes the variable name x from the global scope and decrements the reference count of the object ```DeleteMe``` by 1 (the object which x was referencing). On the other hand, when we reference an object, we increase its reference count by 1. Under the hood, the CPython API has a method ```void Py_DECREF(PyObject *o)``` which decrements the reference count for object o by 1 each time it is called. The CPython garbage collector uses reference counting (maintaining a list of references to all Python objects) to know when to free up space. When an objects reference count hits 0, ```__del__``` is called at the collector knows that it is safe to deallocate space for the object. 

But this still doesnt fully explain why our message was not printed in the second example after we deleted y. When we refenced y by calling it just before deleting it, the REPL actually stored the value of y in another hidden variable: ```_```. This variable ```_``` that the REPL uses references the value of the last non ```None``` expression. In this case, simply referencing y before deleting it gave us a non ```None``` expression (returning the object type and location in memory). When this happened, the CPython method ```void Py_INCREF(PyObject *o)
``` was called, increasing the reference count of the ```DeleteMe``` object by 1. When we called ```globals()```, this caused our hidden REPL variable to change, now storing the dictionary which represents our global variables, thus decreasing the reference count of ```DeleteMe``` from 1 to 0. At this point, ```__del__``` was finally called. This behaviour would not occur outside the REPL. If we ran this as a module, our message would be printed out when we deleted y (like in the first example).  

You can check the reference count of an object manually with ```sys.getrefcount()```. Given what we know about reference counting, try to make sense of the following output:

{% gist 7e5969a1762dc410adce06ab0ec16b0d %}

So the reference count of ```DeleteMe``` in the code above is 6. Let's count the 6 references: 
1.  When we define ```DeleteMe``` on line 1. 
2.  When we access ```DeleteMe``` on line 6.
3.  When we assign ```DeleteMe``` to x on line 6.
4. When we assign x to y.
5. When we call ```sys.getrefcount``` on ```DeleteMe```.
6.  Our hidden REPL variable ```_``` stores the last non ```None``` expression, which references ```DeleteMe```.  
