---
layout: post
title:  "A journey from simple namespaces to object oriented programming"
date:   2021-02-17 10:26:00 -0500
---  
As Python developers we are constantly surrounded by objects in every program we write. Although we are always surrounded by these objects, we don't often stop and conciously consider their importance. How did programming in langauages such as Python become object-oriented to begin with? What factors led to the object-oriented paradigm arising as the star of the show? What are the motivations which naturally compelled developers to begin organizing their code as classes and objects?  

In this post I will be taking the time to reflect on these motivations. We will go on a journey starting from simple namespaces and ending at the object-oriented paradigm. We will consider problems along the way at each step of the journey. Problems which will motivate some kind of solution in order to progress forward as developers. These solutions will build up in sophistication, solving previous problems, but they will also present new problems of their own. Each problem and solution will get us a bit closer the natural end state of object-orientation. And in doing so, we will have a better appreciation classes and objects, why they are so important, and we will understand that object-oriented programming was a natural outcome which resulted from solving big problems developers faced in the past.  

To start, we must consider some ways we can create namespaces in Python. If you're not familiar, namespaces are simply somewhere that we store mappings of names to values. We have a few ways in Python to accomplish this. Firstly, we could use our own dictionaries. Let's create some dictionaries of people and map those people to their favorite bands.

{% gist 10905cff2b84b3a9829529592f6aeae2 %}

We have 3 namespaces above. One for just Beatles fans, one for only Rolling Stones fans, and one which combines the previous two.  

Dictionaries are one way to store name associated to values. But isn't the mapping of names to values what happens every time we declare a variable in Python? If I said ```name = 'nick'``` Python has to store that mapping somewhere. And it turns out that we are always using namespaces in Python, whether we are conscious of it or not, through ```globals()```. This global namespace is where our declared variables are stored and accessed when we reference them somewhere else. And ```globals()``` is a dictionary itself. We can see below that I can reference the variable ```name``` in the global namespace directly by simply calling the variable. But also, I can access that variable by passing it in string format as a key to ```globals()```, like so:
{% gist cc1185bfe3151734842aff45ffbd9bb5 %}  

And infact, I can update ```globals()``` directly and see that change immediately by calling the variable:
{% gist 8cb80968ecb55b46eb99d0bb93c04e3e %}  

As you probably imagined, we need a local namespace for our local scopes inside, say, a function. This local namespace is works the exact same way as ```globals()``` and is called ```locals()```. In the code below, I will locally scope some variables inside of a function and print them out when the funtion is called. 
{% gist 7574e2934efff664913823607744bc12 %}  

So we have seen 3 examples of namespaces in Python so far: user-defined dictionaries, ```globals()```, and ```locals()```. In the case of our own dictionaries, we needed to access values by using square brackets and passing in a key. As for ```globals()```, and ```locals()```, we can reference them directly when we call the variable in the correct scope. Another way we can do namespaces with Python is through ```types.SimpleNamespace```. This time, we reference values in the namespace with dot notation: 
{% gist 7a1ca01d18955a1143d8253c59871a94 %}  

So we can see that namespaces in Python are more or less dictionaries accessed in different ways. What if I wanted to mimick the behaviour of a dictionary ourselfs to dabble in our own namespace creation? 
