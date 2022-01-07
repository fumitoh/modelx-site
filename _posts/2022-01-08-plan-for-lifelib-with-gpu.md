---
layout: post
comments: true
title: "Plan for lifelib with GPU"
author: "Fumito Hamamura"
categories: lifelib
---

![GPU image](/img/2022-01-08/nvidia-g24c404370_1280.jpg)

[lifelib]'s goal is to be a practical tool for actuarial practitioners.
The [basiclife] and [savings] libraries introduced to lifelib in 2021 were designed to be practical enough so that they can be leveraged for real actuarial tasks.
Especially, the parallel models in the libraries, such as [BasicTerm_M], [BasicTerm_ME] and [CashValue_ME], run significantly fast, thanks to [NumPy] and [pandas], even though they don't fully utilize all the CPU cores on a machine.
Speed is not everything, but it's one of the most important factors for actuarial models to be practical and efficient.
That's why it's going to be extremely exciting to make lifelib models run on GPU.

[lifelib]:https://lifelib.io/
[basiclife]:https://lifelib.io/libraries/basiclife/index.html
[savings]:https://lifelib.io/libraries/savings/index.html

[BasicTerm_M]:https://lifelib.io/libraries/basiclife/BasicTerm_M.html
[BasicTerm_ME]:https://lifelib.io/libraries/basiclife/BasicTerm_ME.html
[CashValue_ME]:https://lifelib.io/libraries/savings/CashValue_ME.html
[NumPy]:https://numpy.org/
[pandas]:https://pandas.pydata.org/
[GPU]:https://en.wikipedia.org/wiki/Graphics_processing_unit

[GPU] is an acronym for Graphical Processing Unit.
GPUs are primarily for computer graphics purposes,
but because of their computing power,
they are used for general purposes
in a wide range of areas, such as deep learning and cryptocurrency mining, that require computationally intensive tasks.
GPUs are made available on major cloud computing platforms,
but they are also on consumer PCs.

In comparison to CPUs, GPUs have many cores.
For example, 12th Gen Intel Core i9-12900K, a high-end CPU for consumer PCs has 16 cores, whereas Nvidia's GeForce 1650, 
a midrange GPU for consumer PCs have 896 cores.
Each GPU core is not as powerful as a CPU core,
but GPUs are better than CPUs at such tasks that involve operations on a large homogeneous data set, as GPUs can utilize all the cores simultaneously for such operations by applying each instruction in parallel to the data set.
Obvious examples for such operations are vector and matrix operations on large vectors or matrixes of homogeneous types.

Traditionally, it has been difficult to utilize GPUs for actuarial models,
because the logic of actuarial models is highly heterogenous,
and model points in a model is processed one after another.
However, the parallel models in lifelib have already been designed to
take advantage of NumPy and pandas, which heavily utilize the fast vector and matrix operations offered by NumPy and pandas,
so making the parallel models run on GPUs should not require
fundamental changes.

The planned approach to run the parallel models on GPU is to make full use of
[CuPy] and [RAPIDS cuDF]. CuPy is an open-source array library for GPU-accelerated computing with Python.
CuPy's interface is highly compatible with NumPy and SciPy.
Likewise, cuDF provides objects for GPU-accelerated computing, which are highly compatible with pandas objects.

[CuPy]:https://cupy.dev/
[RAPIDS cuDF]:https://rapids.ai/

 

