---
layout: post
title:  "'x walrus z': when and why should you use assignment expressions?"
date:   2021-02-13 10:26:00 -0500
---  
Assignment expressions, also known as the walrus operator (```x := z ```) was a proposal in PEP 572. The expression above is pronounced "*x walrus z*", and it allows you to assign values to variables as part of larger expressions where, before Python 3.8, you previously could not (we will get into this in detail later). By the way, it's called the walrus operator because it looks like the eyes and tusks of a <a href="https://en.wikipedia.org/wiki/Walrus#/media/File:Pacific_Walrus_-_Bull_(8247646168).jpg" target="_blank">walrus</a>

At the time, the walrus operator was met with a lot of controversy. During the proposal of the walrus operator in PEP 572, some people argued that assignment expressions were bad and not Pythonic. These people really pushed back and claimed that *they* knew what was Pythonic and what was not. "Assignment expressions are NOT pythonic!" they exclaimed loudly. 

When PEP 572 was proposed, Guido Van Rossum was the BDFL (Benevolent Dictator For Life) of Python. And he disagreed with them. He believed that the walrus operator was a good addition and that it was, in fact, very Pythonic. He ought to know... He created the language after all. Guido went ahead and approved PEP 572, including the walrus operator.

But by then the oppinions of the contributing Python developers had added a lot of stress and difficulty to his life. He knew this was not the first time he had faced such opposition, and certaintly not the last time either. He decided it was time to let others handle the job. Guido promptly stepped down as Pythons BDFL the very next morning. 

This makes the walrus operator one of the most interesting and important PEP proposals in Pythons history. So let's look into assignment expressions and make our own conclusion: was Guido right? Or was his opposition right? Is the walrus operator any good?