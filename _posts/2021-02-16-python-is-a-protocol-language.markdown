---
layout: post
title:  "Python is a protocol-based language"
date:   2021-02-16 10:26:00 -0500
---  
In this post I am going to briefly explain a bit of a mental model which helped me think about Python programming when I was first learning the language. 

When creating, extending, and using classes and objects in Python, there is always some top-level protocol which calls some underlying code to handle how the protocol behaves. When new Python developers start coding, they are constantly interacting with these protocols even without realizing at first.   

What I mean by "top-level protocol" is that we write some code to interact with objects we are interested in working with. Perhaps we call an object which runs some code, maybe we initialize a new variable which points to a Python object, maybe we ask the length of an object with the builtin ```len()```, or add two objects with the builtin ```add()```, maybe we want to index into an item with square brackets [], etc. There is some top-level behaviour we perform in our scripts that calls some underyling code which dictates the behaviour of our actions.   

To hide the internal behaviour at first, imagine I created a new class called a ```TalkingDictionary``` which spike back to me every time I interacted with it in some way. Imagine this new class is stored in a file called ```talking_dict.py```. Let's import it, start writing some of these top-level interactions, and see how the protocol behaves. The commented out parts of the code snippets below are the output.
{% gist a9865f8398e88f50e29487688a137879 %}

We can see above that we initialized a new ```TalkingDictionary``` instance and set a few keys (first and last). Each time we did that, we had the dictionary talk back to us letting us know that it was setting the keys to the values we provided.  

Let's try some other interactions, what if we ask for the value of a key just to be sure it was set correctly:
{% gist 3d837024d3377a422368792a0e53ea6a %}

The dictionary talked back to us again and let us know what the value of the first key was when it found it. Let's continue. What if we wanted to print the dictonary:
{% gist 5406a22deada7cf59c8ee86df92a95ce %}

This is an example of using a builtin function ```print``` to interact with our new object. This is another example of that top-level protocol I was talking about. Let's look at another protocol. Let's ask if the key 'first' is contained in the dictionary.
{% gist 1a24376ced4b846e4c8a847a33a3323e %}

We know 'first' is in the dictionary, we set it just a moment ago. But according to our talking dict, it's not in there. This is because we overloaded a special method of our ```TalkingDictionary``` to return False all the time no matter what the user asks for. In fact, we have overloaded numerous special methods whic dictate the behaviour of the protocol. Let's try asking for a key that is not in the dictionary. 
{% gist 09c64e5e3ebfa4285b35de2f32ed2c81 %} 
 
When we ask for the value of the key 'middle', which is not contained in our dictionary, you can tell that at some point it is calling the same code which was called when we accessed the key 'first' because it prints out the message "Let me find that for you...". And just after a ```KeyError``` is raised with the message "You're looking for 'middle' but it ain't here." So at least 2 distinct peices of code were called in this interaction even though we only asked for one key.   

What is we tried to call our talking dictionary as if it were a function.
{% gist 1538a187dff04baa64ed392ba558ad85 %}

Notice that simply adding ```()``` to the end of our instance is itself one of these top-level protcols between the user and the underlying class. These ending braces are calling code behind the scenes.   

Finally, let's use on more builtin function and ask for the length of our dictionary.
{% gist 3ab0c33418856969bad65a49c8693bd0 %} 

So we can see that each of our interactions with this ```TalkingDictionary``` class has some associated behaviour which is predetermined by a protocol (the code which links our actions to whatever Python outputs back to us). It turns out that all of these interactions, getting a key, setting a key, asking if a key is in the dictionary, printing the instance, asking for its length, ect., are calling methods under the namespace of the ```TalkingDictionary``` which explain why our instance behaves as it does. I defined all of these methods by inheriting from ```dict```, the basic Python dictionary, and overloading the behaviour myself.   

Let's look at our ```TalkingDictionary``` class: 

{% gist 4c202b6286239d0d8d5b8182c3c4c434 %}

Looking at the class above, you can see which methods handle the top-level protocols we initiated in our examples. Setting a new key, like we did when we set 'first' to 'Nick' called ```__setitem__()``` which printed a message and then called the same method from its parent, ```dict```, to actually set the key, value pair. When we asked for the value of 'first', our instance called ```__getitem__()``` which did a similar thing. ```print()``` called ```__repr__()```. Using the ```in``` operator called ```__contains__()```. Attempting to access a key which did not exist, 'middle', called ```__getitem()__``` first, which printed out the message "Let me find that for you..." and then when it discovered that middle was not a key in our dictionary, it called ```__missing__()``` which raised a ```KeyError```. When we called our instance by adding ```()``` to the end, it called ```__call__()```. And finally, when we called ```len()``` on our instance, it called the method ```__len__()```.  

If this is completely new to you, you might wonder why these special methods have leading and trailing double underscores. In Python, these special methods are called "dunders" (double underscore) and are methods whose novel implementation is strictly reserved for the core Python developers, but are supposed to be overloadded by the user. In other words, never implement your own dunder method, but feel free to alter their behaviour in the classes you create if neccessary.   

These methods and protocols, and many more, are explained in detail in the <a target="_blank" href="https://docs.python.org/3/reference/datamodel.html">Python Data Model docs.</a>

I hope this helps new Python developers arrive at a clear mental model for this protocol-based nature of Python.