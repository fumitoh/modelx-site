---
layout: post
title: "Heads-up on lifelib development"
author: "Fumito Hamamura"
categories: lifelib
---

[lifelib]: https://lifelib.io

It’s been a while since I made a major update on [lifelib],
so I want to give some heads-up on what I’ve been planning to reflect in the next and future versions of [lifelib]. Below are the items I’ve been working on or plan to tackle in near future.

## From script-based to model-based

All [lifelib] models, except for the [smithwilson](https://lifelib.io/projects/smithwilson.html) model, which is the most recent addition to [lifelib], are built from Python scripts and Python modules. This is going to change from the next release. modelx is now capable of saving models into text files as pseudo Python modules under a folder tree. [lifelib] projects from the next release contain *model* folders, and models are read from there. The Python scripts and modules will continue to be available in a sub-folder named *scripts*.


## Making existing models simpler

I have some ideas to simplify existing models to make the models faster to load, easier to understand and more transparent. The ideas rely on modelx features that are yet to be implemented. One of the ideas is to simplify input loading. Currently the input data for each model point is read into a Space object from an Excel file, which is an expensive operation. This can be simplified by reading the Excel range containing the data for all model points as one object.
Another idea is to simplify the `Space.formula` property. The property is used to pass references to child dynamic spaces, but the expression can be quite complex when the Space has child Spaces.
modelx specs need to be changed to achieve this, and I’m now thinking about how to achieve relatively referencing one Space from another.


## Introducing **parallellife**

A modelx user suggested an approach to build a fast model using numpy or pandas with modelx. The idea is to use numpy arrays or pandas Series as Cells values, instead of values of primitive types, such as `float` and `int`. This idea has already been applied to the [smithwilson](https://lifelib.io/projects/smithwilson.html) model, which has Cells returning numpy arrays.  
When this technique is applied to building life projection models, the formula of each Cells calculates values not for one model points, but for many model points in parallel. Comparing to an ordinary model where calculations are carried out for one model point at a time, this model should gain extreme performance improvement, as most calculations are passed to naively-compiled numpy modules, and numpy can utilize multi-threading and hardware level parallelization.
I was not aware that modelx could be used this way until I was told by him, but Cells can contain arbitrary types of values, so this is a valid use of modelx.
I haven’t started on this model yet, but this is among those with the highest priority.  The downside of this parallel model is, it cannot be the same model as a serial model, one that calculates values for one model point at a time, such as the [simplelife](https://lifelib.io/projects/simplelife.html) model, because Cells operations in the parallel model needs to be written in terms of vector operations in stead of scalar operations. Both vector and scalar operations should be identical in most places, but not all the places.
