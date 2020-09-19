---
layout: post
title: "modelx v0.10.0 will make lifelib simpler"
author: "Fumito Hamamura"
categories: modelx
---

modelx v0.10.0 introduces relative reference, a new way of referencing modelx objects in setting values of derived References. This post explains how relative References can help simplify modelx models.

The code below shows the formula property of the `Projection` space in the simplelife model from lifelib v0.14.0.

```python
>>> simplelife.Projection.formula
def _formula(PolicyID, ScenID=1):
    refs = {'pol': Policy[PolicyID],
            'asmp': Assumption[PolicyID],
            'scen': Economic[ScenID],
            'DiscRate': Economic[ScenID].DiscRate}
    return {'refs': refs}
```

The formula has always been that convoluted since the first release of lifelib and I wanted to make it simpler and easier to understand. I had thought about how to enhance modelx so that the formula above could be stripped down to a simpler expression. After a long while I finally came up with the idea of relative reference and implemented in modelx v0.10.0, which was released a couple days ago. The concept of relative and absolute reference is somewhat to similar to that of spreadsheets.

With the introduction of relative reference, the formula can be as simple as below.

```python
>>> newlife.Projection.formula
lambda PolicyID, ScenID=1: None
```

## How relative reference works

Let’s say you want to build a simple cashflow projection model of life insurance policies, one that's similar to but much simpler than simplelife.

You would create a Space to put Cells for projection calculations. Let’s name the Space `Projection` here.  You could put all the Cells needed for projection in `Projection`, but to avoid filling the Space with too many Cells,  you would decide to create 2 child spaces `Policy` and `Assumptions`, and put Cells representing policy attributes in `Policy`, and put Cells representing assumptions in `Assumptions`.

![Spaces](/img/2020-09-19/spacetree.png)

For example, Projection would contain `PolsDeath`, Number of Death, which need to refer to `BaseMortRate` in its formula. `BaseMortRate` is in `Assumptions`, and in turn it needs to refer to `Product` in `Policy`.

`PolsDeath` could refer to `BaseMortRate` in its formula as `Assumptions.BaseMortRate`, but it’s too verbose, so you decide to define an alias for `Assumptions` as a Reference, and name it `asmp`, then you get to refer to `BaseMortRate` in `PolsDeath`’s formula as `asmp.BaseMortRate`.

In order for `BaseMortRate` to be able to refer to `Product` in `Policy`,  `Product` needs to be visible from `Assumptions`. One way to achieve this is to define a Reference that is bound to `Product`. Let’s name it `prod` here.

![Base Space](/img/2020-09-19/basespace.png)

Then you want to parameterize `Projection` by `PolicyID`, so that the projection for each model point can be expressed as `Projection[PolicyID]`, such as `Projection[1]`, `Projection[2]`, etc..

You would think that setting the parameter as below would work:

```python
>>> Projection.parameters = ("PolicyID",)

>>> Projection.formula
lambda PolicyID: None
```

But this does not work with modelx v0.9.0 or older, because
in `Projection[1]`, derived References, such as `asmp` and `prod` reference the same objects as what `asmp` and `prod` reference, which are in the base Space, `Projection`.

![Absolute reference](/img/2020-09-19/absref.png)

With modelx v0.10.0, this behavior is changed.

When References are set by the assignment operation, the References are set in auto mode. In this example, `asmp` and `prod` are auto References. When `Projection[1]` explicitly inherits from `Projection`, `asmp` and `prod` in `Projection[1]` are derived from `asmp` and `prod` in `Projection`. Since `Projeciton.asmp` and `Projection.prod` are auto References, their derived References, `Projection[1].asmp` and `Projection[1].prod` are bound to objects in `Projections[1]`, whose relative names in `Projection[1]` are consistent with the relative names of the values of the base References in `Projection`.

If `Projeciton.asmp` or `Projction.prod` were bound to Spaces or Cells outside `Projection`, then `Projection[1].asmp` and `Projection[1].prod` would reference the same Spaces or Cells as the values of the base References. If `asmp` and `prod` were set in relative mode, then creating `Projection[1]` would raise an Error, because relative reference would be impossible.

![Relative reference](/img/2020-09-19/relref.png)

## lifelib will become simpler


With the introduction of relative reference, lifelib models will be simpler. I haven’t updated the lifelib package yet, but I developed an experimental implementation of an updated simplelife as [newlife][1]. A refined version of the newlife will replace the current simplelife.

[1]:{{site.url}}/download/2020-09-19/newlife.zip

In the current version (0.0.14) of the simplelife model, `Projection` space has the following formula as I mentioned earlier.

```python
>>> simplelife.Projection.formula
def _formula(PolicyID, ScenID=1):
    refs = {'pol': Policy[PolicyID],
            'asmp': Assumption[PolicyID],
            'scen': Economic[ScenID],
            'DiscRate': Economic[ScenID].DiscRate}
    return {'refs': refs}
```

The `Policy` and `Assumption` Spaces are located outside the `Projection` Space, and the formula brings in their dynamic sub Spaces `Policy[PolicyID]`, `Assumption[PolicyID]` in `Projection[PolicyID]` by defining References `pol` and `asmp` and binding them to `Policy[PoicyID]` and `Assumption[PolicyID]`.

The [new model][1] has `Policy` and `Assumptions` (renamed from `Assumption`) as child Spaces of `Projection`.  In `Projection`, `asmp` and `pol` are defined as auto References and bound to `Policy` and `Assumptions` to be used as aliases for them. In `Assumptions`, auto References `prod`, `polt`, `gen`, `sex` are defined and reference `Product`, `PolicyType`, etc.., which are Cells in `Policy`.

![newlife](/img/2020-09-19/newlife.png)

And `Projection.formula` is as simple as:

```python
lambda PolicyID, ScenID=1: None
```

Thanks to relative reference, when `Projection[1]` is created, `pol` and `asmp` get bound to `Projection[1].Policy` and `Projection[1].Assumptions`, and `prod`, `polt`, etc..., in `Projection[1].Assumptions` get bound to `Projection[1].Policy.Product`, `Projection[1].Policy.PolicyType`, etc...
