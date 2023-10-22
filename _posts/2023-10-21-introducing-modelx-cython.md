---
layout: post
comments: true
title: "Introducing modelx-cython: Boosting modelx with Cython Compilation"
categories: modelx
---

## modelx's Push Towards Native-Code Performance

[The recent addition of the export feature]({% post_url 2023-07-29-export-feature-intro %}) has substantially enhanced modelx's capabilities. As evidenced in [our previous post]({% post_url 2023-08-19-enhanced-speed-for-exported-lifelib-models %}), exported models greatly benefit from a notable performance boost.

However, the journey of modelx doesn’t end here. When comparing models purely written in Python (without numpy assistance) to their native-code equivalents, the former significantly lags in speed. Typically, for computationally demanding tasks, compiled languages like C or C++ are favored. Thus, our next milestone for modelx is to create native-code models that far outpace the speed of exported models.

To this end, we're introducing *modelx-cython*, an experimental python package. This tool transforms exported modelx models into native code using the prowess of Cython.

## Demystifying Cython

Cython is a well-established tool that accelerates Python code by translating it to C, and subsequently compiling it to native code using an external C compiler. With over two decades of development, foundational Python libraries like numpy, pandas, scipy, and scikit-learn leverage Cython to build their C extension modules.

While Cython can translate nearly all Python code to C, a direct translation doesn't always yield significant performance enhancements. This limitation stems from the use of Python objects in the translated C code, even for primitive objects like integers. To truly harness performance benefits, original Python code needs tweaking.

Cython extends the Python language with its own set of syntax, which allows users to optimize Python code by defining static types for variables and functions, as depicted in the following illustration:

![Image1](/img/2023-10-22/cython-to-nativecode.png)

However, a newer Cython feature lets users adjust the original Python code using pure Python syntax, leveraging decorators and Python’s type annotation feature. This enables Cython to generate C code where objects, especially numeric ones, are represented as C types, ensuring performance levels close to those written originally in compiled languages.

## Introducing modelx-cython

modelx-cython, our new experimental package, creates native-code models from exported modelx models.

In the visualization below, modelx-cython handles the latter half of the conversion, while modelx's export functionality manages the former half:

![Image2](/img/2023-10-22/modelx-ecosystem.png)

modelx-cython converts a pure-Python model exported by modelx into native code. This process comprises two primary steps:

1. **Preparing Cython Source Files**:
    - Initially, modelx-cython executes a sample script you provide to gather run-time type information.
    - It then creates a copy of the original pure-Python model, appending "_cy" to the model's name.
    - With the type data and a spec file you supply, modifications are made to the source files of the copied model. This is done to optimize them for Cython compilation, emphasizing the translation into as many static objects as possible using the newly added type details. 
    - In addition, a setup file is generated for Cython invocation in the next step.

2. **Cython Compilation**:
    - Cython then compiles these prepared source files.
    - Upon successful compilation, binary files (with extensions *.pyd for Windows and *.so for Linux) are produced alongside the source files. As a result, when you import this "_cy" version of the model into Python, Python reads these faster, compiled binary modules.


## A lifelib Example


**A Walkthrough Using the *BasicTerm_S* Model**

To demonstrate the process, we'll use the *BasicTerm_S* model from the *basiclife* library in lifelib. For reference, all code snippets provided here are available in the `example` directory of the [modelx-cython GitHub repository](https://github.com/fumitoh/modelx-cython).

1. **Exporting the Model**:
   
   Start by exporting the *BasicTerm_S* model using modelx:

   ```python
   import modelx as mx
   mx.read_model("BasicTerm_S").export("BasicTerm_S_nomx")
   ```

2. **Setting Up the Sample File**:
   
   You'll need to create a sample Python script, which will be executed in the initial step to capture run-time type information. Below is an example of this sample file:

   ```python
   from BasicTerm_S_nomx import BasicTerm_S
   
   for i in range(1, 11):
       BasicTerm_S.Projection[i].pv_net_cf()
   ```

   Change the current directory to the location where *BasicTerm_S_nomx* is located.
   Then, create a `sample.py` file and populate it with the provided code.

3. **Preparing the Spec File**:

   In addition to the sample file, you also need a spec file. This file is for stipulating supplementary configuration settings not sourced from the sample run. In this example, the nested `dict` expression below determines that the parameter `t` in the `Projection` space has a size of `241` and that the return type for `disc_factors` should be of the `object` type, irrespective of the type designation from the sample run:

   ```python
   {"spaces": {"Projection": {"cells_params": {"t": {"size": 241}},
                              "cells": {"disc_factors": {"return_type": "object"}}}}}
   ```

   In the same directory, create a `spec.py` file and insert the code snippet above.

4. **Invoking `mx2cy`**:

   modelx-cython installs the `mx2cy` command. Invoke the `mx2cy` command from your terminal:

   ```
   mx2cy BasicTerm_S
   ```

   On successful completion, a new package labeled *BasicTerm_S_nomx_cy* will be created alongside *BasicTerm_S_nomx*. Within this directory, you'll identify the compiled Python module binary files (*.pyd for Windows or *.so for Linux) produced during the `mx2cy` operation.


## Performance Comparison

Let's measure the performance of our compiled model against its pure Python counterpart. The following script computes net cashflow present values for 10,000 model points and aggregates them.

```python
import sys
import os
import timeit
import psutil

is_nomx = '--nomx' in sys.argv
is_cy = '--cython' in sys.argv

process = psutil.Process(os.getpid())
def get_mem_use():
    """Return the memory usage of the current process in MB"""
    return process.memory_info().rss / (1024 ** 2)

if is_nomx:
    from BasicTerm_S_nomx import BasicTerm_S
elif is_cy:
    from BasicTerm_S_nomx_cy import BasicTerm_S
else:
    raise RuntimeError("Parameter missing")

result = {}
def run_model(m, size):
    mem_use = get_mem_use()
    result['value'] = sum(m.Projection[i].pv_net_cf() for i in range(1, size+1)) / size
    result['mem_use'] = get_mem_use() - mem_use

result['time'] = timeit.timeit(
    'run_model(BasicTerm_S, 10000)', number=1, globals=globals())

print(result)
```

The summarized performance results on both Windows and Linux (Windows Subsystem for Linux) are:

**Run Time (sec.)**

| Model | Windows | Linux |
| ------ | -------- | ------ |
| Pure-Python | 17.7 | 14.1 |
| Cythonized | 7.8 | 6.4 |
| Ratio | 2.3 | 2.2 |


**Memory Usage (MB)**

| Model | Windows | Linux |
| ------ | -------- | ------ |
| Pure-Python | 1831 | 1736 |
| Cythonized | 440 | 443 |
| Ratio | 4.2 | 3.9 |

On both platforms, the compiled models run at double the speed of their pure-Python counterparts and consume only one-fourth of the memory compared to the pure-Python models.

## What's Next?

Though promising, the current release of modelx-cython is experimental and may not work with many modelx models without manual tweaks.

Despite our compiled model being over twice as fast as the pure-Python version, we believe a tenfold speed increase is attainable with native code. The reason the potential performance isn't fully realized is that while Cython compiles numeric objects (like integers and floats) into primitive C types, other objects, such as lists and pandas objects, remain as Python objects. As a result, the generated binary code relies on the Python interpreter to call these objects' methods. To circumvent this inefficiency, the original modelx formulas should be written with Cython's translation and compilation performance in mind. Alternatively, modelx-cython could analyze the original formulas for users, identify inefficient segments, and then adjust them for optimal compilation.

It’s a challenging path ahead, but one we're excited to traverse. We envision modelx evolving into the premier tool for building full-fledged production models.


