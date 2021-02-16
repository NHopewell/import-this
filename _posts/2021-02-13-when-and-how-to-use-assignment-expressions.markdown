---
layout: post
title:  "'x walrus z': when and why should you use assignment expressions?"
date:   2021-02-13 10:26:00 -0500
---  
Assignment expressions, also known as the walrus operator (```x := z ```) was a proposal in <a href = "https://www.python.org/dev/peps/pep-0572/" target="_blank">PEP 572</a> by . The expression above is pronounced "*x walrus z*", and it allows you to assign values to variables as part of larger expressions where, before Python 3.8, you previously could not (we will get into this in detail later). By the way, it's called the walrus operator because it looks like the eyes and tusks of a <a href="https://en.wikipedia.org/wiki/Walrus#/media/File:Pacific_Walrus_-_Bull_(8247646168).jpg" target="_blank">walrus</a>

At the time, the walrus operator was met with a lot of controversy. During the proposal of the walrus operator in PEP 572, some people argued that assignment expressions were bad and not Pythonic. These people really pushed back and claimed that *they* knew what was Pythonic and what was not. "Assignment expressions are NOT pythonic!" they exclaimed loudly. 

When PEP 572 was proposed, Guido Van Rossum was the BDFL (Benevolent Dictator For Life) of Python. And he disagreed with them. He believed that the walrus operator was a good addition and that it was, in fact, very Pythonic. He ought to know... He created the language after all. Guido went ahead and approved PEP 572, including the walrus operator.

But by then the oppinions of the contributing Python developers had added a lot of stress and difficulty to his life. He knew this was not the first time he had faced such opposition, and certaintly not the last time either. He decided it was time to let others handle the job. Guido promptly stepped down as Pythons BDFL the very next morning. 

This makes the walrus operator one of the most interesting and important PEP proposals in Pythons history. So let's look into assignment expressions and make our own conclusion: was Guido right? Or was his opposition right? Is the walrus operator any good?

As previously stated, the walrus operator allows the developer to assign values to variables in places where assignment were previously not allowed. The most common being within an ```if``` statement. Let's look at some common patterns and use cases which could benefit from the walrus operator. The first common pattern involves any circumstance we need to read from a variable, check if its value is non-zero, and then use that value. Let's pretend we work at a burrito shop and we are serving customers. First, let's initialize the current stock of our different burritos:
```python
burrito_stock = {
    'beef': 20,
    'chicken': 16,
    'black_bean': 11,
    'veggie': 27
}
```
Let's say a customer comes in and wants to buy a beef burrito. First, we need to check if we have a beef burrito to sell, and if we do, we can check out her/his order. We could do this by getting count of our current stock of beef burritos and then checking for a non-zero value with an ```if``` statement:
```python
def make_beef_burrito(count):
    ...
def out_of_stock():
    ...

count = burrito_stock.get('beef', 0)
if count:
    make_beef_burrito(count)
else:
    out_of_stock()

```
We can see this code may be misleading because it's easy to assume that the count variable is accessed in both the ```if``` and ```else``` branches. Because count is declared at the top, but is only accessed in the ```if``` branch, we might mistake it for being more important than it actually is. 

Assignment expressions make this exact scenario cleaner. Here is the same example with the walrus operator:
```python
if count := burrito_stock.get('beef', 0):
    make_beef_burrito(count)
else:
    out_of_stock()

```
Now it becomes very clear where the count variable is being used. It is more obvious at a glance that count is relevant to the ```if``` branch. This two step process of assigning and then evaluating is the core concept behind the walrus operator. 

If we continue with this scenario, but instead of simply doing a non-zero check, we let the customer order multiple beef burritos, say 3, we now need to do a comparison inside the ```if``` statement. Without the walrus operator, that would look like this:
```python
size_order = 3
count = burrito_stock.get('beef', 0)
if count >= size_order:
    make_beef_burrito(count)
else:
    out_of_stock()

```
This still presents the same problem of over emphasizing the count variable. We can clean this up again by making it clear where count is important with the walrus operator:
```python
size_order = 3
if (count := burrito_stock.get('beef', 0)) >= size_order:
    make_beef_burrito(count)
else:
    out_of_stock()

```
It is important to note that when our assignment expression is a subexpression of a larger expression, like above, we need to wrap it in parentheses. 







One caveat, mentioned by Python core developer Raymond Hettinger in the docs <a href="https://docs.python.org/3/whatsnew/3.8.html" target="_blank">'What's New in Python 3.8'</a>, is that we should:
> Try to limit use of the walrus operator to clean cases that reduce complexity and improve readability.  

So don't go wild with the walrus. Be sensible. Be Pythonic.