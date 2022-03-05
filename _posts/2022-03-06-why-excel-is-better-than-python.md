---
layout: post
comments: true
title: "Why Excel is better than Python"
categories: modelx
---

Actuaries, quants and other mathmatical finance experts use Excel a lot. 
They know it's not a good practice to use Excel for complex modeling, but they keep using it anyway.

Excel is bad because it's error prone. 
Numbers are manually input in small cells in large sheets.
Formulas have nested parentheses and refer to other cells by addresses, which makes it hard to understand them.
Workbooks are hard to compare each other and not ideal for version-control.   

Despite all of that, actuaries stick to Excel. 
Some try to introduce Python, or whatever programming language they like, aiming to replace Excel with it.
However, in most cases, such initiatives fail,
because the users are so much in love with Excel, 
and without exploring much, they think Python is not a good fit for their jobs.

Excel's popularity is often attributed to it's prevalence and ease of learning.
But we want to think deeper here, 
because the reason why modelx was born is to bring Excel's advantages into Python.
What is it that makes people feel comfortable using Excel over Python?

## Python runs, Excel stays

A Python program runs. It runs from the top of the script file through the end of it. Before it runs, it's just a text file. 
The program exists for just a short period of time while it's running, and when it finishes, it vanishes and goes back to a text file again.

In contrast, Excel doesn't run. it doesn't even have a "Run" button. It persists. it's static, 
in a way that it always holds the results of your formulas.

Which gives you peace of mind more, a thing that has to be running to exist and to get your request done, 
or a thing that stays still and always holds the results for you?


## Loop or recursion

How do you calculate present value of cashflows in Python?
You would do something like:

```python
pv = 0
for i in range(10):
    pv += cashflow[i] / (1 + 0.5)**(i + 1)
```
Or if you're more Pythonic, then your code would be more like:

```python
pv = sum(cashflow[i] / (1 + 0.5)**(i + 1) for i in range(10))
```

Either way, the same logic is looped and intermediate values,
such as `pv` at `i=5` or `cashflow[i] / (1 + 0.5)**(i + 1)`,
are not remembered at the end of the loop.

Now, let's see how you would do it in Excel:

![Excel](/img/2022-03-26/pv-in-excel.png)

As you see on the left, Excel uses recursion instead of loop, to repeat the same formula, 
so `pv` for all the `i` are available. On the right are a different way of taking the present value.
In this case, `cashflow[i] / (1 + 0.5)**(i + 1)` for all the `i` are preserved.
Excel remembers and presents the intermediate values used for obtaining the end result.
Excel lets you inspect intermediate values and trace formula dependencies, which
allows you to feel more comfortable with the result's accuracy.

## Formula-value coupling

In Excel, a formula and its value are in the same cell. They are always coupled.
As long as Excel is in the automatic calculation mode,
you are certain that the value is always up to date and any updates on the formula are reflected.

In Python, a piece of code and its result are separated.
In the case of the example above, the result is stored in the variable `pv`.
`pv` and it's code, for example, `sum(cashflow[i] / (1 + 0.5)**(i + 1) for i in range(10))`, exist separately. And even if the code is updated, `pv` holds the old value.
You need to explicitly update `pv`, for example, by rerunning the entire script.


## Single entry exit

A python program has a single entry point and single exit point.
Python interprets a script file from the top of the file through the bottom.
Usually, a part of the entire program does not run individually without it's context.

On the other hand, an Excel file has no explicit entry point and exit point,
because it does not "run". So you don't need to care about
the order of calculations. You don't need to be think about which formula should be
evaluated before which. Excel resolves the order for you. 
If the order is invalid, Excel detects the cyclic reference. 

## Object-oriented

Excel is object-oriented out of the box. You don't even notice that you're building a model in an objected-oriented way.
You don't need to define classes and methods on your own,
because objects, such as workbooks, worksheets and ranges are already provided by Excel for you to start defining formulas.
This lets you break down the entire logic into small pieces of formulas.
Without even noticing, you follow the best practice of splitting large logic into components.

In Python, you would be likely to end up writing hundreds of lines of code in one script file without defining any classes or functions.
It's hard to start from defining class and functions from scratch on a plain text file without proper experience. Even if you're capable of doing, the chances are, your colleagues in your team are not.


## modelx design

modelx was invented to address those issues. 
Models in modelx work like Excel workbooks.
A modelx model does not run. It stays. It keeps all the intermediate values so you can 
trace the formulas and values.
Formulas and their values are coupled. Models don't have specific entry points or end points. Models are object-oriented. You don't need to write class or methods.
This is why modelx's catchphrase is "Use Python like a spreadsheet!". 




