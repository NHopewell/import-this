---
layout: post
title:  "A journey from simple namespaces to object oriented programming"
date:   2021-02-17 10:26:00 -0500
---  
As Python developers we are constantly surrounded by objects in every program we write. Although we are always surrounded by these objects, we don't often stop and conciously consider their importance. How did programming in langauages such as Python become object-oriented to begin with? What factors led to the object-oriented paradigm arising as the star of the show? What are the motivations which naturally compelled developers to begin organizing their code as classes and objects?  

In this post I will be taking the time to reflect on these motivations. We will go on a journey starting from simple namespaces and ending at the object-oriented paradigm. At each step in this journey, we will build on the previous by asking what problems might arise with our current implementation, and eventually come to the conclusion that object-oriented programming is a *natural solution* to all of these problems. We will also get a much better appreciation for the relevance of dictionaries in Python along the way.

### First things first: namespaces in Python

To start, we must consider some ways we can create namespaces in Python. If you're not familiar, namespaces are simply somewhere that we store mappings of names to values. The important thing to remember with namespaces is that *one namespace is never enough*. In fact, sometimes we need a lot of namespaces to organize our logic. We have a few ways in Python to accomplish namespaces.  

Firstly, we could use our own dictionaries. Let's create some dictionaries of people and map those people to their favorite bands.

{% gist 10905cff2b84b3a9829529592f6aeae2 %}

We have 3 namespaces above. One for just Beatles fans, one for only Rolling Stones fans, and one which combines the previous two.  

Dictionaries are one way to store name associated to values. But isn't the mapping of names to values what happens every time we declare a variable in Python? If I said ```name = 'nick'``` Python has to store that mapping somewhere. And it turns out that we are always using namespaces in Python, whether we are conscious of it or not, through ```globals()```. This global namespace is where our declared variables are stored and accessed when we reference them somewhere else. And ```globals()``` is a dictionary itself. We can see below that I can reference the variable ```name``` in the global namespace directly by simply calling the variable. But also, I can access that variable by passing it in string format as a key to ```globals()```, like so:
{% gist cc1185bfe3151734842aff45ffbd9bb5 %}  

And infact, I can update ```globals()``` directly and see that change immediately by calling the variable:
{% gist 8cb80968ecb55b46eb99d0bb93c04e3e %}  

As you probably imagined, we need a local namespace for our local scopes inside, say, a function. This local namespace is works the exact same way as ```globals()``` and is called ```locals()```. In the code below, I will locally scope some variables inside of a function and print them out when the funtion is called. 
{% gist 7574e2934efff664913823607744bc12 %}  

Now we have seen 3 examples of namespaces in Python so far: user-defined dictionaries, ```globals()```, and ```locals()```. In the case of our own dictionaries, we needed to access values by using square brackets and passing in a key. As for ```globals()```, and ```locals()```, we can reference them directly when we call the variable in the correct scope. Another way we can do namespaces with Python is through ```types.SimpleNamespace```. This time, we reference values in the namespace with dot notation: 
{% gist 7a1ca01d18955a1143d8253c59871a94 %}  

So we can see that namespaces in Python are more or less dictionaries accessed in different ways. What if I wanted to mimick the behaviour of a dictionary ourselfs to dabble in our own namespace creation?    

### Iteration one: creating our own dictionary behaviour manually with arrays

If we want to create our own mappings of names to values, then we will need to create some buckets for names (or keys) and some buckets for values. To create some buckets, we could create list of lists of size ```size``` to represent keys (where each imbedded list represents its own bucket). And we could do the same for values.  
{% gist fc64d9e5777b286b0fae1c3537607ba7 %}


We will also need some way to connect the key buckets and value buckets. To do this, we could simply ```hash()``` the key % size (the length of the lists). This will return an integer in the range of 0 to size-1 we can use to index into our buckets. Here is what that might look like:
{% gist ecae1ec3956f3f524ec5904273a3b72d %}  

Now we can manually hash keys and values into buckets at the same index of these two lists. So we have a hacky dictionary we can play with.   

**Problems with this implementation:**  
- Our key and value buckets are both stored in the global namespace which can quickly become cluttered with variable names. Recall that one namespace is never enough.  
- We have to manually create each peice of this solution (creating buckets, hashing to those buckets, and then appending to those buckets). 
- No modularity or extensibility in this implementation.   

An obvious next step would be to try to avoid this manual nature of our current implementation by moving logic into functions. We want to have functions to set and get items from our namespace.  

### Iteration two: creating our own dictionary behaviour with functions  

Our main steps to creating and using our own namespace is to 1) initialize the key and value buckets, 2) set items at the same index within the key and value buckets by hashing the key, and 3) getting an item from the value bucket based on the key we pass. To implement and use these three steps as functions, we would move the code into functions as so:
{% gist 66b1d800cfb85ec96518ba16167b2290 %}  

We can see by looking at the code within the ```if __name__ == '__main__':``` block that we now have a functional way to create something which behaves like a dictionary. This is an improvement over the purely manual approach but is far from perfect.   

**Problems with this implementation:**  
- We are still relying on the global namespace and so a variable name can only be used once. 
- If we continue this way, we will pollute the global namespace quickly and run into name collisions.  

The core issue is that one namespace is not enough. A potential sollution would be to take this same functional approach we laid out above, but instead of relying on the global namespace, we initialize a smaller namespace within globals, a new dictionary, and populate that instead.

### Iteration three: creating our own dictionary behaviour with subdictionary namespaces   

You will see in the code below that the only thing I am really changing is that I am creating a new namespace each time I want to model the behaviour of a dictionary, storing values such as ```size```, ```key_buckets```, and ```value_buckets``` in that namespace instead of ```globals()```. Notice that ```initialize_buckets()``` now takes a namespace which already has a size stored. ```set_item()``` and ```get_item()``` also take this namespace as an argument. 
{% gist b7c3b5cf246119f7a9a5e98fecd10230 %}

With this implementation, everytime we initialize our buckets we are creating our own namespace in the form of a new dictionary. This makes polluting the global namespace more difficult. This is what ournamespace looks like:
{% gist a882dcb741b4709e14e187f0e7d1c7fc %}  

Although we now have the ability to work with many namespaces, this approach still isn't perfect (albeit very useful).

**Problems with this implementation:**  
- Clunky, we ideally do not want to access values from each of our namespaces like we do with dictionaries (square bracket notation). 
- Not Pythonic: if our code is constantly filled with brakcets and strings, it will be ugly. Does this matter in Python? Absolutely. The very first line of the Zen of Python states "beautiful is better than ugly".  

A better way would be to use dot notation. So instead of saying ```namespace["key"]``` we would like to say ```namespace.key```.  

### Iteration four: creating our own dictionary behaviour with SimpleNamespace  

Again the code will barely change. Now, instead of using a dictionary everytime we want to create a namespace, we will use SimpleNamspace instead. Recall that SimpleNamespace let's use use this dot notation we want.
{% gist e296dbe105ad8670895007c2f7e23386 %}  

Now the notation of accessing values in our namespace is much nicer. But you might have noticed that this solution is not fully automated. In fact, there are a couple glarring problems: 

**Problems with this implementation:**  
- We need to manually create new SimpleNamespaces every time we need a namespace.  
- We need to pass that namespace to each single function call, causing a lot of repetitive code.   

We will now see that the solution to this problem is to use Python classes.

### Iteration five: creating our own dictionary behaviour with our own Python class  

When writing the same example as a Python class, notice that all I am changing is that the indentation is different and I am passing ```self``` instead of the namespace (like I did before). ```self``` refers to the instance itself, which I called "namespace". There is no place where I am initializing a builtin dict nor a SimpleNamespace. The variable "namespace" becomes the actual namespace directly this time. Our class builds a new namespace for us when we instantiate it.   

{% gist 2e85e2be29123a5289ba861efba4f8a3 %}  

Notice above that instead of passing the namespace as the first argument to each function call, when I use the class methods above (accessed with dot notation) the namespace is passed implicitly via ```self```. We do not need to actually specify the namespace directly every time. Everything except for this detail is identical to the functional implementation which came before it.   

Although this implementation is great, it is not perfect.   

**Problems with this implementation:**  
- We have to call method names every time to set and get items.   

Ideally, when we initialize our buckets, set items in our buckets, and get items from our buckets, we do not want to write the entire method name out. We want this to happen automatically and share a common protocol with other objects. 

### Iteration six: creating our own dictionary behaviour with dunder methods and operator protocols  

If you are not familiar with the operator protocol of Python, I highly recommend you read a previous post I made which explains it: <a href="https://nhopewell.github.io/import-this/2021/02/16/python-is-a-protocol-language.html" target="_blank"><strong>Python is a protocol-based language</strong></a>. It is important to understand this protocol behaviour to understand how dunder methods work.  

Essentially, we want to overload some special methods which the Python core developers have implemented so we can interact with Pythons operator protocol (called "dunders" because they are surrounded by double underscores). In our case, ```initialize_buckets()``` will become the dunder method ```__init__()```, ```set_item()``` will become ```__setitem__()```, and ```get_item()``` will become ```__getitem()__```. To understand more about these special items, read about them in the <a target="_blank" href="https://docs.python.org/3/reference/datamodel.html">Python Data Model docs.</a>

{% gist 649d46669216f762788abab764d3a426 %}  

You can now see that we are directly recreating the builtin dict by mimicking the behaviour of initializing, as well as getting and setting new items. The behaviour Python expects the user to input in order to create, mutate, and access our new class is exactly the same as what is expected by real dictionaries. *This* consistency is what makes it an operator protocol.  

We now have solved all of our previous problems and have a very consistent interface. And it turns out, this system comes with a lot of great side effects we won't discuss here. By following each iteration, we solved some problems and created new ones. Slowly we built up to the final solution in iteration six. This solution is how Object-Oriented programming is implemented in Python. Hopefully this style of programming now feels natural to you and you have a better appreciation for it.  

You also hopefully have a better appreciation for how poweful dictionaries are in Python. We can completely mimick the behaviour of classes and methods with dictionaries and functions. You might think it's strange to use dictionaries like this at first. But in Python, most important things are dictionaries. Like we mentioned before, ```globals()```, ```locals()```, and ```SimpleNamespace``` are all dictionaries. And as we have seen, **classes in Python are just thin layers which sit ontop of dictionaries.** The actual module I am writing this code in can be represented as a dictionary itself. If you have ever run a debugger on a python module you'll see that special variables such as the name of the file, its location, its package name etc., are all stored as key, value pairs. 
