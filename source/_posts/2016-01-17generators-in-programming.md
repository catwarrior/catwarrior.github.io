title: generators in programming
date: 2016-01-17 11:23:37
tags:
- stuff
---
I'm reading python recently, the reason is, it is popular and wildly used, and it's also muture.

There is concept "generators" in python, which is used to compute a value at runtime.
ex.
{codeblock}
list = [x for x in range(1,100)]
{endcodeblock}
In fact, this will create a list of 100 items with values, most of the time it's ok, but when item number
is very large, or even unpredictable, you cannot do this.
So here comes the generators.

{codeblock}
g = (x for x in range(1,100))
{endcodeblock}
when you type `g` in the py commandline, it will tell you `g` is a generator type, you can call `next(g)` to
iterate the data members.

you could even create your function act as a generator, the key is the keyword `yield`.

{codeblock}
def odd():
    print('step 1')
    yield 1
    print('step 2')
    yield(3)
    print('step 3')
    yield(5)
{endcodeblock}

Actually, it's common concept in programming languages, for example, C# also have `yield`, exactly the same. 
