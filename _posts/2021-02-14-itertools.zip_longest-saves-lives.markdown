---
layout: post
title:  "itertools.zip_longest() saves lives"
date:   2021-02-14 10:26:00 -0500
---  
We know how great ```zip()``` is for processing multiple iterators in parrallel. If I have a list of names and another list of the
lengths of those names, I can iterate over both lists at once like so:
{% gist da23e55bcd0e52dfedf3739c8254dc0c %}

One important issue which can come up when using ```zip()``` is that it stops yeilding results once it reaches the length of the smallest list. So if I add another name to the names list, but don't add an accompanying length to the names_lengths list, ```zip()``` will stop once it has exhausted the names_lengths list:
{% gist de5311e94c35e384c95e13439e42e1c0 %}
Where is Sarah? Is she gone? did ```zip()``` kill Sarah?!?!  

This behaviour is fine most of the time because we often want to iterate through one list along with a second list which was derived from the first (such as a list of lengths). But sometimes we want to iterate through both until we exhaust the longest list (not killing poor Sarah).

```itertools.zip_longest()``` comes to the rescue in these situations. It yeilds results until it exhausts the longest list. Here is zip_longest at work with our lists of different lengths:
{% gist c01e0150376a6c8ed5be3029fa14cf25 %}
And there we go, ```itertools.zip_longest()``` saved Sarah!  

Notice above that we probably don't want to print "sarah is length None". Sarah does have a length, Python just doesn't know what it is in this situation. In these cases, we want to provide a value to the optional argument *fillvalue*, which defaults to None.
{% gist e2e7e28242953cb7469fdbc35d5be604 %}
```itertools.zip_longest()``` is roughly equivalent to:
{% gist 83cd83811662a4c7f9a5cbb8ad2a16ae %}
