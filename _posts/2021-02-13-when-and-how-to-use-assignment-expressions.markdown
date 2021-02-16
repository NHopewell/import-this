---
layout: post
title:  "'y walrus z': when and why should you use assignment expressions?"
date:   2021-02-15 10:26:00 -0500
---  
Assignment expressions, also known as the walrus operator (```y := z ```) was a proposal in <a href = "https://www.python.org/dev/peps/pep-0572/" target="_blank">PEP 572</a> by . The expression above is pronounced "*y walrus z*", and it allows you to assign values to variables as part of larger expressions where, before Python 3.8, you previously could not (we will get into this in detail later). By the way, it's called the walrus operator because it looks like the eyes and tusks of a <a href="https://en.wikipedia.org/wiki/Walrus#/media/File:Pacific_Walrus_-_Bull_(8247646168).jpg" target="_blank">walrus</a>

At the time, the walrus operator was met with a lot of controversy. During the proposal of the walrus operator in PEP 572, some people argued that assignment expressions were bad and not Pythonic. These people really pushed back and claimed that *they* knew what was Pythonic and what was not. "Assignment expressions are NOT pythonic!" they exclaimed loudly. 

When PEP 572 was proposed, Guido Van Rossum was the BDFL (Benevolent Dictator For Life) of Python. And he disagreed with them. He believed that the walrus operator was a good addition and that it was, in fact, very Pythonic. He ought to know... He created the language after all. Guido went ahead and approved PEP 572, including the walrus operator.

But by then the oppinions of the contributing Python developers had added a lot of stress and difficulty to his life. He knew this was not the first time he had faced such opposition, and certaintly not the last time either. He decided it was time to let others handle the job. Guido promptly stepped down as Pythons BDFL the very next morning. 

This makes the walrus operator one of the most interesting and important PEP proposals in Pythons history. So let's look into assignment expressions and make our own conclusion: was Guido right? Or was his opposition right? Is the walrus operator any good?

As previously stated, the walrus operator allows the developer to assign values to variables in places where assignment were previously not allowed. The most common being within an ```if``` statement. Let's look at some common patterns and use cases which could benefit from the walrus operator. By common patterns I mean scenarios which frequently come up during development.   

These patterns are as follows:    

* **pattern 1:** reading from a variable, checking its value is non-zero, and using that value in some way.
* **pattern 2:** assigning a value to a variable based on some condition, and then referencing that variable in a function call right after. 
* **pattern 3:** writing nested multiple ```if```, ```elif```, and ```else``` statements in place of Pythons non-existent switch/case statement.
* **pattern 4:** constructing an ugly "*loop-and-a-half*" in place of Pythons non-existent do-while loop.

### Pattern # 1
The first common pattern involves any circumstance we need to read from a variable, check if its value is non-zero, and then use that value. Let's pretend we work at a burrito shop and we are serving customers. First, let's initialize the current stock of our different burritos:
```python
burrito_stock = {
    'beef': 20,
    'chicken': 16,
    'black_bean': 11,
    'veggie': 27,
    'tofu': 8
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
    beef_burrito = make_beef_burrito(count)
else:
    out_of_stock()
```
We can see this code may be misleading because it's easy to assume that the count variable is accessed in both the ```if``` and ```else``` branches. Because count is declared at the top, but is only used in the ```if``` branch, we might mistake it for being more important than it actually is. 

Assignment expressions make this exact scenario cleaner. Here is the same example with the walrus operator:
```python
if count := burrito_stock.get('beef', 0):
    beef_burrito = make_beef_burrito(count)
else:
    out_of_stock()
```
Now it becomes very clear where the count variable is being used. It is more obvious at a glance that count is relevant to the ```if``` statement. This two step process of assigning and then evaluating is the core concept behind the walrus operator. 

If we continue with this scenario, let's pretend to make one veggie burrito we need 2 rations of veggies instead of just one. We would now need to do a comparison inside the ```if``` statement. Without the walrus operator, that would look like this:
```python
def make_veggie_burrito(count):
    ...


count = burrito_stock.get('veggie', 0)
if count >= 2:
    veggie_burrito = make_veggie_burrito(count)
else:
    out_of_stock()

```
This still presents the same problem of over emphasizing the count variable. We can clean this up again by making it clear where count is important with the walrus operator:
```python
if (count := burrito_stock.get('veggie', 0)) >= 2:
    veggie_burrito = make_veggie_burrito(count)
else:
    out_of_stock()
```
It is important to note that when our assignment expression is a subexpression of a larger expression, like above, we need to wrap it in parentheses. In any case where we do not need the parentheses, they should be dropped entirely. This goes for any subexpression in Python, not only ones which contain the walrus operator.  

### Pattern # 2 
The second common pattern, which is similar to the first pattern, involves anytime we want to assign a value to a variable depending on some condition, and then use that variable in a function call right after. Let's continue with the veggie burrito customer. Let's say in order to fulfill the order I need the 2 portions of veggies (like before) to be sliced up, and if we don't have enough to slice, I want to raise an OutOfVeggies exception. Here is a way to implement this scenario without the walrus operator:
```python
def make_veggie_burrito(slices):
    ...

def slice_veggies(count):
    ...

class OutOfVeggies(Exception):
    pass


slices = 0
count = burrito_stock.get('veggie', 0)
if count >= 2:
    slices = slice_veggies(count)

try:
    veggie_burrito = make_veggie_burrito(slices)
except:
    raise OutOfVeggies:
        out_of_stock()
```
Now we can see the potential confusion really begin to arise. We see slices and count initialized above the branching logic, and at first glance we might think they are both important to all the logic below. In reality, only slices is important outside the ```if``` condition, count is only used in the call to ```slice_veggies()```. Let's rework with with the walrus operator:
```python
slices = 0
if (count = burrito_stock.get('veggie', 0)) >= 2:
    slices = slice_veggies(count)

try:
    veggie_burrito = make_veggie_burrito(slices)
except:
    raise OutOfVeggies:
        out_of_stock()
```
Now it is much more clear that slices is relevant to all the logic below, and count is only important to the ```if``` statement. At a glance, we understand the scope of importance and any room for ambiguity has gone away. 

### Pattern # 3  
The third common pattern involves scenarios where a switch/case statement might be a programmers first choice when tackling some sub problem they face, but she/he cant' use one because Python does not support switches (as of yet). Typically, Python developers handle this with nesting multiple ```if```, ```elif```, and ```else``` statements.

Let's pretend that we want to automate buritto making for our non-eat eating customers. We can see that 3 of our options in our stock do not contain meat. These are veggies, black beans, and tofu burritos. Let's say that we know veggie is the most popular, then black bean, then tofu. And each of these options requires a different amount of each ingredient to make. Our system might look like this:
```python
count = burrito_stock.get('veggie', 0)
if count >= 2:
    slices = slice_veggies(count)
    burrito = make_veggie_burrito(slices)
else:
    count = burrito_stock.get('black_bean', 0)
    if count >= 3:
        burrito = make_black_bean_burrito(count)
    else:
        count = burrito_stock.get('tofu', 0)
        if count:
           burrito = make_tofu_burrito(count)
        else:
           burrito = "No burrito for you, bosss."
```
This type of code is common in Python because of the lack of a switch statement. But when we clean this up with the walrus operator, it almost feels like we are suddenly using a switch statement:
```python
if (count := burrito_stock.get('veggie', 0)) >= 2:
    slices = slice_veggies(count)
    burrito = make_veggie_burrito(slices)
elif (count := burrito_stock.get('black_bean', 0)) >= 3:
    burrito = make_black_bean_burrito(count)
elif (count := burrito_stock.get('tofu', 0)):
    burrito = make_tofu_burrito(count)
else:
    burrito = "No burrito for you, bosss."
```
We have stripped out all the nesting statements and it is obvious now what is happening in this bit of code. In situations where ugly nested branching logic is showing up in your code, it is probably worth your time to consider reimplementing the logic using assignment expressions.

That being said, it was recently announced that <a target="_blank" href="https://github.com/gvanrossum/patma">switch statements will be coming to Python soon</a>. This construct has been approved as of <a href="https://www.python.org/dev/peps/pep-0634/" target="_blank">PEP 634</a>. It seems the Python switch will be much more powerful than switch implementations in other languages like C#. If you're reading in the future, maybe this has already been released. I imagine these proper switch statements will be the preferred solution to this common pattern we just discussed.

### Pattern 4  
The fourth and final pattern involves any scenario a Python developer writes a "loop and a half" in their code to make up for the lack of a do-while loop in the language. 

Let's say I want to make burritos until I am out of ingredients:
```python
def get_ingredients():
    ...

def make_burritos_batch(base, count):
    ...

burritos = []
ingredients = get_ingredients()               # call get_ingredients()
while ingredients:
    for base, count in ingredients.items():
        burrito_batch = make_burrito_batch(base, count)
        burritos.extend(burrito_batch)
    ingredients = get_ingredients()          # call get_ingredients() again
```
The obvious repetition above involves the two calls to ```get_ingredients()```. We call it once before the while loop and then again at the end of the while loop to restock the ingredients.  

One common way Python developers try to avoid this repetition is to use a "*loop-and-a-half*". This strategy will get rid of the repetition but will make the while loop a simple infinite loop controlled by a ```break``` statement. Something we try to avoid:
```python
burritos = []
while True:                                   # loop
    ingredients = get_ingredients() 
    if not ingredients:                       # and a half
        break
    for base, count in ingredients.items():
        burrito_batch = make_burrito_batch(base, count)
        burritos.extend(burrito_batch)
```
We can avoid this awkward inifite loop with a break strategy with the walrus operator. Because the walrus operator allows us to assign a value in a conditional check, we do not need to initize ingredients before the loop, calling ```get_ingredients() ``` twice. Instead we can reassign the ingredients variable and check it at each pass of the loop:
```python
burritos = []
while ingredients := get_ingredients():                                   
    for base, count in ingredients.items():
        burrito_batch = make_burrito_batch(base, count)
        burritos.extend(burrito_batch)
```
We can see in this example that it's hard to argue that the walrus operator did not clean up this code. This is clean and Pythonic and does not require any ugly tricks.

Now we have seen 4 common patterns where the walrus operator comes in handy. So far, we are making a really strong case for assignment expressions. That being said, perhaps you are not convinced yet that there is real value in the walrus. I want to go over one more important area where the walrus operator is a nice addition. 

### Using the walrus operator in comprehensions to keep code DRY

We all know what DRY means: Don't Repeat Yourself. The idea is that if we are copy and pasting code, we probably can clean up our code in some way to avoid this. Let's say I am selling flowers at a flower shop and customers are buying flowers in batches of 8. To sell the flowers in these batches, I first want to see how many batches of 6 I can support from my total flow stock for each flower ordered:
```python
flower_stock = {
    'rose': 42,
    'tulip': 28,
    'sunflow': 7,
    'orchid': 12
}

flower_order = ['tulip', 'sunflower', 'orchid']

def get_flower_batches(count, size):
    return count // size

total_batches = {}
for flower in flower_order:
    count = flower_stock.get(flower, 0)
    num_batches = get_flower_batches(count, 8)
    if num_batches:
        total_batches[flower] = num_batches

print(total_batches)

# {'tulip': 3, 'orchid': 1}
```
We could instead attempt write this more Pythonically with a dictionary comprehension:
```python
total_batches = {
    flower: get_flower_batches(flower_stock.get(flower, 0), 8)
    for flower in flower_order
    if get_flower_batches(flower_stock.get(flower, 0), 8)
}
```
This code really is not better because the call to ```get_flower_batches()``` happens twice and it takes away from the readability of the code overall. This code would also not act as expected if we changed the actual parameters input to one of the two function calls but not the other.

The walrus operator can solve this problem:
```python
total_batches = {
    flower : batches for flower in flower_order
    if (batches := get_flower_batches(flower_stock.get(flower, 0), 8))
}
```
Now our code looks really DRY. But is it better? To a long time Python developer the answer is probably yes. To someone who is new to Python the answer might be no. This might look too cryptic to someone who is not used to this syntax.

Calling back to the original discussion about who was right about assignment expressions being a good addition, Guido or the critics, let's think about it. We saw that there a lot of instances where the walrus operator does improve readability. We could tell in our branching examples that it was clear how to mentally scope some of our variables at a glance when we incorporated the walrus operator. And in our dictionary comprehension example, we can see that the walrus operator helped us keep our code DRY bur some less experienced Python devs might find the resulting code a bit cryptic. 

For me personally, I think Gudio was right. It is a good additon *when used correctly*.  One caveat, mentioned by Python core developer Raymond Hettinger in the docs <a href="https://docs.python.org/3/whatsnew/3.8.html" target="_blank">'What's New in Python 3.8'</a>, is that we should:
> Try to limit use of the walrus operator to clean cases that reduce complexity and improve readability.  

So there are places we ought to make the most of the walrus, but don't go wild with it either. Be sensible. Be Pythonic.