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

### Step one: creating our own dictionary behaviour manually with arrays

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

### Step two: creating our own dictionary behaviour with functions  

Our main steps to creating and using our own namespace is to 1) initialize the key and value buckets, 2) set items at the same index within the key and value buckets by hashing the key, and 3) getting an item from the value bucket based on the key we pass. To implement and use these three steps as functions, we would move the code into functions as so:
{% gist 66b1d800cfb85ec96518ba16167b2290 %}  

We can see by looking at the code within the ```if __name__ == '__main__':``` block that we now have a functional way to create something which behaves like a dictionary. This is an improvement over the purely manual approach but is far from perfect.   

**Problems with this implementation:**  
- We are still relying on the global namespace and so a variable name can only be used once. 
- If we continue this way, we will pollute the global namespace quickly and run into name collisions.  

The core issue is that one namespace is not enough. A potential sollution would be to take this same functional approach we laid out above, but instead of using the global namespace, we initialize a smaller namespace within globals, a new dictionary, and populate that instead.

### Step three: creating our own dictionary behaviour with subdictionary namespaces   

The key difference in this step is to reduce the reliance on the global namespace. You will see in the code below that the only change I am really making is that I am creating one new namespace each time I want to model the behaviour of a dictionary, storing values such as ```size```, ```key_buckets```, and ```value_buckets``` in that namespace instead of ```globals()```.



With this implementation, everytime we initialize our buckets we are creating our own mini namespaces in the form of new dictionaries. This makes polluting the global namespace more difficult.   


You might think it's strange to use dictionaries like this at first. But in Python, most important things are dictionaries. Like we mentioned before, ```globals()```, ```locals()```, and ```SimpleNamespace``` are all dictionaries. And as we have seen, **classes in Python are just thin layers which sit ontop of dictionaries.** The actual module I am writing this code in itself can be represented as a dictionary. If you have ever run a debugger on a python module you'll see that special variables such as the name of the file, its location, its package name etc., are all stored as key, value pairs. 
