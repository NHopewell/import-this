---
layout: post
title:  "'y walrus z': when and why should you use assignment expressions?"
date:   2021-02-15 10:26:00 -0500
---  
Assignment expressions, also known as the walrus operator (```y := z ```) was a proposal in <a href = "https://www.python.org/dev/peps/pep-0572/" target="_blank">PEP 572</a> by . The expression above is pronounced "*y walrus z*", and it allows you to assign values to variables as part of larger expressions where, before Python 3.8, you previously could not. By the way, it's called the walrus operator because it looks like the eyes and tusks of a <a href="https://en.wikipedia.org/wiki/Walrus#/media/File:Pacific_Walrus_-_Bull_(8247646168).jpg" target="_blank">walrus</a>

At the time, the walrus operator was met with a lot of controversy. During the proposal of the walrus operator in PEP 572, some people argued that assignment expressions were bad and not Pythonic. These people really pushed back and claimed that *they* knew what was Pythonic and what was not. "Assignment expressions are NOT pythonic!" they exclaimed loudly. 

When PEP 572 was proposed, Guido Van Rossum was the BDFL (Benevolent Dictator For Life) of Python. And he disagreed with them. He believed that the walrus operator was a good addition and that it was, in fact, very Pythonic. He ought to know... He created the language after all. Guido went ahead and approved PEP 572, including the walrus operator.

But by then the oppinions of the contributing Python developers had added a lot of stress and difficulty to Guidos life. He knew this was not the first time he had faced such opposition, and certaintly not the last time either. He decided it was time to let others handle the job. Guido promptly stepped down as BDFL the very next morning. 

This makes the walrus operator one of the most interesting and important PEP proposals in Pythons history. So let's look into assignment expressions and make our own conclusion: was Guido right? Or was his opposition right? Is the walrus operator any good?

As previously stated, the walrus operator allows the developer to assign values to variables in places where assignment was previously not allowed. The most common being within an ```if``` statement. Let's look at some common patterns and use cases which could benefit from the walrus operator. By common patterns I mean scenarios which frequently come up during development.

### Pattern # 1
The first common pattern involves any circumstance we need to read from a variable, check if its value is non-zero, and then use that value. Let's pretend we work at a burrito shop and we are serving customers. First, let's initialize the current stock of our different burritos:
{% gist e41ee3542a5ab8226e4ed8cced150a3b %}

Let's say a customer comes in and wants to buy a beef burrito. First, we need to check if we have a beef burrito to sell, and if we do, we can make it up for her/him to eat. We could do this by getting count of our current stock of beef burritos and then checking for a non-zero value with an ```if``` statement:
{% gist 9581865e2e2c5d035dfadcc9de5a8464 %}

We can see this code may be misleading because it's easy to assume that the count variable is accessed in both the ```if``` and ```else``` branches. Because count is declared at the top, but is only used in the ```if``` branch, we might mistake it for being more important than it actually is. 

Assignment expressions make this exact scenario cleaner. Here is the same example with the walrus operator:
{% gist 261c4f717941c71442dc3ae35ee1a73b %}

Now it becomes very clear where the count variable is being used. It is more obvious at a glance that count is relevant to the ```if``` statement. This two step process of assigning and then evaluating is the core concept behind the walrus operator. 

If we continue with this scenario, let's pretend to make one veggie burrito we need 2 rations of veggies instead of just one. We would now need to do a comparison inside the ```if``` statement. Without the walrus operator, that would look like this:
{% gist dc4a6742e08db9e3f1ae9c9e505f1ad4 %}

This presents the same problem of over emphasizing the count variable. We can clean this up again by making it clear where count is important with the walrus operator:
{% gist eeebf5f5599d18738282d62bc64bea81 %}

It is important to note that when our assignment expression is a subexpression of a larger expression, like above, we need to wrap it in parentheses. In any case where we do not need the parentheses they should be dropped entirely. This goes for any subexpression in Python, not only ones which contain the walrus operator.  

### Pattern # 2 
The second common pattern, which is similar to the first pattern, involves anytime we want to assign a value to a variable depending on some condition, and then use that variable in a function call right after. Let's continue with the veggie burrito customer. Let's say in order to fulfill the order I need the 2 portions of veggies (like before) to be sliced up, and if we don't have enough veggies to slice, I want to raise an OutOfVeggies exception. Here is a way to implement this scenario without the walrus operator:
{% gist bfe070d4b296e7690b0ddd739ac61c56 %}

Now we can see the potential confusion really begin to arise. We see slices and count initialized above the branching logic, and at first glance we might think they are both important to all the logic below. In reality, only slices is important outside the ```if``` condition, count is only used in the call to ```slice_veggies()```. Let's rework with with the walrus operator:
{% gist fb27ee15f5983580c9555aa6de1e47cf %}
Now it is much more clear that slices is relevant to all the logic below the line where it is declared, and count is only important to the ```if``` statement. At a glance, we understand the scope of importance of these two variables and any room for ambiguity has gone away. 

### Pattern # 3  
The third common pattern involves scenarios where a switch/case statement might be a programmers first choice when tackling some sub problem they face, but she/he cant' use one because Python does not support switches (as of yet). Typically, Python developers handle this with nesting multiple ```if```, ```elif```, and ```else``` statements.

Let's pretend that we want to automate buritto making for our non-meat eating customers. We can see that 3 of our options in our stock do not contain meat. These are veggie, black bean, and tofu burritos. Let's say that we know veggie is the most popular, then black bean, then tofu. And each of these options requires a different amount of each ingredient to make. Our system might look like this:
{% gist ea7983fd6619edc506c829d17a9dfc99 %}

This type of nested code is quite common in Python because of the lack of a switch statement. But when we clean this up with the walrus operator, it almost feels like we are suddenly using a switch statement:
{% gist f08f86453470301aac49c8d11c2e50c1 %}

We have stripped out all the nesting statements and it is obvious now what is happening in this bit of code. In situations where ugly nested branching logic is showing up in your code, it is probably worth your time to consider reimplementing the logic using assignment expressions.

That being said, it was recently announced that <a target="_blank" href="https://github.com/gvanrossum/patma">switch statements will be coming to Python soon</a>. This proposal has been approved as of <a href="https://www.python.org/dev/peps/pep-0634/" target="_blank">PEP 634</a>. It seems the Python switch will be much more powerful than switch implementations in other languages like C#. If you're reading in the future, maybe this has already been released. I imagine these proper switch statements will be the preferred solution to this common pattern we just discussed.

### Pattern # 4  
The fourth and final pattern involves any scenario a Python developer writes a "*loop-and-a-half*" in their code to make up for the lack of a do-while loop in the language. 

Let's say I want to make burritos until I am out of ingredients:
{% gist 406471b828a3f7f132f2cb5c3316764e %}

The obvious repetition above involves the two calls to ```get_ingredients()```. We call it once before the while loop and then again at the end of the while loop to either restock the ingredients or break out of the loop.  

One common way Python developers try to avoid this repetition is to use a "*loop-and-a-half*". This strategy will get rid of the repetition but will make the while loop a simple infinite loop controlled by a ```break``` statement. Something we try to avoid. It is called a loop and a half because we execute half the code in the loop before exiting, instead of checking a proper condition first, thus never starting the terminal iteration to begin with.
{% gist 22367d27a0cb1efbd0b9f6bf8c939eb1 %}
 
We can avoid this awkward inifite loop with a break strategy with the walrus operator. Because the walrus operator allows us to assign a value in a conditional check, we do not need to initize ingredients before the loop, calling ```get_ingredients() ``` twice. Nor do we need to implement a forever loop with a break statement. Instead we can reassign the ingredients variable and check it at each pass of the loop:
{% gist 17f0ada3883002c679e8a43e53ad6b9b %}

We can see in this example that it's hard to argue that the walrus operator did not clean up this code. This is clean and Pythonic and does not require any ugly tricks.

Now we have seen 4 common patterns where the walrus operator comes in handy. So far, we are making a really strong case for assignment expressions. That being said, perhaps you are not convinced yet that there is real value in the walrus. I want to go over one more important area where the walrus operator is a nice addition. 

### Using the walrus operator in comprehensions to keep code DRY

We all know what DRY means: Don't Repeat Yourself. The idea is that if we are copy and pasting code, we probably should clean it up. Let's say I am selling flowers at a flower shop and customers are buying flowers in batches of 8. To sell the flowers in these batches, I first want to see how many batches of 8 I can support from my total stock for each flower ordered:
{% gist 4d23303f93c12e82d52351598a0252c6 %}

We could instead attempt write this more Pythonically with a dictionary comprehension:
{% gist 57959f44c6c0dbd259388671ca11741c %}

This code really is not much better because the call to ```get_flower_batches()``` happens twice and it takes away from the readability of the code overall. This code would also not act as expected if we changed the actual parameters input to one of the two function calls but not the other.

The walrus operator can solve this problem:
{% gist 716a930c9be1f8c9375ab79f19b4d52e %}

Now our code looks really DRY. But is it better? To a long time Python developer the answer is probably yes. To someone who is newer to Python the answer might be no. This might look too cryptic to someone who is not used to this syntax.  

---  
<br>


Calling back to the original question: who was right about assignment expressions being a good addition, Guido or the critics? Let's make a conclusion. We saw that there a number of instances where the walrus operator does improve readability. We could tell in our branching examples that it was clearer how to mentally scope some of our variables at a glance when we incorporated the walrus operator. And in our dictionary comprehension example, we can see that the walrus operator helped us keep our code DRY. On the other hand, some devs might find code which makes use of assignment expressions a bit too cryptic. They might say that such code is not what 'Pythonic' code is supposed to read like.

For me personally, I think Gudio was right. It is a good additon *when used correctly*.  One caveat, mentioned by Python core developer <a target="_blank" href="https://twitter.com/raymondh?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor">Raymond Hettinger</a> in the docs <a href="https://docs.python.org/3/whatsnew/3.8.html" target="_blank">'What's New in Python 3.8'</a>, is that we should:
> Try to limit use of the walrus operator to clean cases that reduce complexity and improve readability.  

So in conclusion: there are places we ought to make the most of the walrus, but don't go wild with it either. Be sensible. Be Pythonic.