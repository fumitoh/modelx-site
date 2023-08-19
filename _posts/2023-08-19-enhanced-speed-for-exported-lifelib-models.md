---
layout: post
comments: true
title:  "Enhanced Speed for Exported lifelib Models"
categories:
    - modelx
    - lifelib
---

## Introduction

In the previous post, the experimental feature of modelx version 0.22.0 was highlighted, detailing its capability to export models as self-contained Python packages. However, many modelx models in lifelib couldn't be exported without manual adjustments.

With the release of modelx 0.23.0, the export functionality has undergone significant enhancement, moving away from its initial experimental phase.
Additionally, the models in lifelib have been updated to remove any deprecated expressions. These updates are now part of the latest release of lifelib, version 0.9.2.

Using modelx 0.23.0 now allows for the seamless export of most models from the recent lifelib 0.9.2 libraries. The exported models are self-contained Python packages, completely independent of modelx.

The previous post hinted at the possibility that exported models might showcase superior performance compared to the original versions. After conducting preliminary tests with lifelib models, the outcomes appear to be truly impressive.

## Exportable lifelib Models

With modelx v0.23.0, all modelx models in the following libraries in lifelib 0.9.2, including their example models, can be exported:

- basiclife
- savings
- assets
- economic
- smithwilson (project)

However, modelx models from older libraries (previously known as "projects" excluding the smithwilson project), remain non-exportable.

## Understanding modelx Models: Seriatim vs. Vectorized

Before diving into the test results, it's important to understand that there are two primary modeling approaches within modelx: the seriatim and the vectorized models.


**1. Seriatim Models:**
A seriatim model processes each model point on each scenario individually.
In the case of an insurance cashflow model, a space in the model, or a tree of spaces would represent cashflows, assumptions, and policy attributes of one model point under one deterministic scenario, and a new space is created for each combination of model point and scenario, .

*Example:* Consider `BasicTerm_S` in basiclife. Within it, the `Projection` space represents a cashflow projection for one model point. So, for every model point, a dynamic copy of `Projection`, such as `Projection[1]`, is generated. 
This seriatim modeling approach is widely accepted in many industry models.


**2. Vectorized Models:**
Vectorized models work differently. They don't process scenarios and model points individually. Instead, the logic within these models operates on vectors, where formulas take in and return values as vectors for all model points across all scenarios simultaneously.

*Example:* In the `BasicTerm_M` vectorized model, if there are 10,000 model points for a single scenario, the `premiums` formula within `Projection` gives back a 10,000-element vector, with each element signifying the amount for an individual model point at time `t`.


Now, you might wonder, if both `BasicTerm_S` and `BasicTerm_M` yield identical outcomes, why have two distinct approaches? It's because each has its advantages and drawbacks.

**Speed & Efficiency:** 
Vectorized models, like `BasicTerm_M`, are markedly faster. A test with 10,000 scenarios showed that while `BasicTerm_M` finished in seconds, `BasicTerm_S` took several minutes. This speed difference arises because `BasicTerm_M` uses efficient libraries written in C like numpy and pandas for computations, whereas `BasicTerm_S` relies on the Python interpreter. Moreover, the overhead from modelx, which offers enhanced modeling features like dependency tracing, can slow down seriatim models and impact memory consumption.

**Flexibility & Application:** 
However, vectorized models aren't always the ideal solution. Many actuarial projection models are too intricate for the vectorized approach, making it nearly impossible to implement. For example, modeling requirements where projections from both the current and future points in time are needed can be challenging for vectorized models. Often, most real-world actuarial models prefer the seriatim approach due to its flexibility.

Given these differences, the main challenge is enhancing the performance of seriatim models. This goal led to the introduction of the export feature.
Primarily, the export feature was introduced to benefit seriatim models, as removing the modelx overhead would have a more pronounced positive effect on them compared to vectorized models. And as we'll discuss next, our test results confirm this expectation.


## Test Results and Analysis


For the testing, Python 3.11.4 and a Windows machine with the specifications detailed below was utilized. Its 64GB memory capacity was enough for the test cases examining up to 10,000 model points.

- CPU: 12th Gen Intel Core i7-12700KF 3.60 GHz
- Memory Size: 64GB

Creating exported models is a straightforward process. For instance, the code below will generate an exported model named `BasicTerm_S_nomx` in the current directory from the modelx model `BasicTerm_S`, which is also located in the same directory.

```python
import modelx as mx
mx.read_model("BasicTerm_S").export("BasicTerm_S_nomx")
```


### Seriatim Models Testing

For this test, the performance of `BasicTerm_S` from `basiclife` was compared between the original modelx and its exported version.

The test code measured the time taken, memory usage, and ensured consistent results between the two models by checking the average present value of net cashflows.

```python
import os
import timeit
import psutil

is_mx = True    # Set False for testing exported models

process = psutil.Process(os.getpid())
def get_mem_use():
    """Return the memory usage of the current process in MB"""
    return process.memory_info().rss / (1024 ** 2)

if is_mx:
    import modelx as mx
    BasicTerm_S = mx.read_model('BasicTerm_S')
else:
    from BasicTerm_S_nomx import BasicTerm_S

result = {}
def run_model(m, size):
    mem_use = get_mem_use()
    result['value'] = sum(m.Projection[i].pv_net_cf() for i in range(1, size+1)) / size
    result['mem_use'] = get_mem_use() - mem_use

result['time'] = timeit.timeit(
    'run_model(BasicTerm_S, 1000)', number=1, globals=globals())

print(result)
```

The table below summarizes the results with 1000 model points.

|   | modelx | Exported | Ratio |
| -- | -- | -- | -- |
|Run Time (second) | 25.42 | 1.87 | 13.58 |
|Memory Usage (MB) | 2800.64 | 179.38 | 15.61 |


The exported model was impressively faster, taking under 2 seconds, which is 14 times quicker than the original. It also consumed only 1/16th of the memory.

The same test was conducted using 10,000 model points, 10 times the size used in the above.

|   | modelx | exported | Ratio |
| -- | -- | -- | -- |
|Run Time (second) | 291.67 | 18.44 | 15.82 |
|Memory Usage (MB) | 29496.98 | 1830.91 | 16.11 |

The exported model finished in less than 20 seconds, outperforming the original by being 16 times faster and using only 1/16th of the memory. The original model used a considerable amount of memory due to its need to maintain a large dependency graph
for tracking the dependencies of formula calls during the run.


### Testing with vectorized models

Next, the vectorized models were put to the test.
First, `BasicTerm_M` with 10,000 model points was compared against its exported model.
The test code for this model was slightly modified from the seriatim version.  

```python
import os
import timeit
import psutil

is_mx = True    # Set False for testing exported models

process = psutil.Process(os.getpid())
def get_mem_use():
    """Return the memory usage of the current process in MB"""
    return process.memory_info().rss / (1024 ** 2)

if is_mx:
    import modelx as mx
    BasicTerm_M = mx.read_model('BasicTerm_M')
else:
    from BasicTerm_M_nomx import BasicTerm_M

result = {}
def run_model(m):
    mem_use = get_mem_use()
    result['value'] = m.Projection.result_pv()['PV Net Cashflow'].mean()
    result['mem_use'] = get_mem_use() - mem_use

result['time'] = timeit.timeit(
    'run_model(BasicTerm_M)', number=1, globals=globals())

print(result)
```

The table below summarizes the results of the test.

|   | modelx | exported | Ratio |
| -- | -- | -- | -- |
|Run Time (second) | 0.43 | 0.36 | 1.21 |
|Memory Usage (MB) | 207.94 | 205.01 | 1.01 |

Here, the exported model was 1.2 times faster than the original. The performance boost is moderate, because each formula is executed just once for all the model points, resulting in a considerably smaller dependency graph.

Another vectorized model, `CashValue_ME_EX4` and its exported model were
also tested. This model by default operates on 9 model points and 1,000 scenarios, totaling 9,000 iterations.  

```python
import os
import timeit
import psutil

is_mx = True    # Set False for testing exported models

process = psutil.Process(os.getpid())
def get_mem_use():
    """Return the memory usage of the current process in MB"""
    return process.memory_info().rss / (1024 ** 2)

if is_mx:
    import modelx as mx
    CashValue_ME_EX4 = mx.read_model('CashValue_ME_EX4')
else:
    from CashValue_ME_EX4_nomx import CashValue_ME_EX4

result = {}
def run_model(m):
    mem_use = get_mem_use()
    m.Projection.scen_size = 1000
    result['value'] = m.Projection.result_pv()['Net Cashflow'].mean()
    result['mem_use'] = get_mem_use() - mem_use

result['time'] = timeit.timeit(
    'run_model(CashValue_ME_EX4)', number=1, globals=globals())

print(result)
```

The table below summarizes the results of the test.

|   | modelx | exported | Ratio |
| -- | -- | -- | -- |
|Run Time (second) | 0.24 | 0.18 | 1.36 |
|Memory Usage (MB) | 332.48 | 330.90 | 1.00 |

The exported model was about 35% faster, with memory consumption being almost identical to the original.

The same test was conducted with 10,000 scenario. 

|   | modelx | exported | Ratio |
| -- | -- | -- | -- |
|Run Time (second) | 1.62 | 1.54 | 1.05 |
|Memory Usage (MB) | 3292.80 | 3291.67 | 1.00 |

The memory usage, compared to the previous test, increased almost exactly 10 times, which is consistent with the increase in number of
scenarios. The exported model was marginally 5% faster than the original model.
The increase in number of scenarios mostly impacted the calculations within pandas and numpy, thus the run time improvement was modest.

## Further Considerations

Through this performance test, 
we found that the exported seriatim models run significantly faster than their original modelx models.
Even though our test models were simpler than real-world production models, the noticeable improvement in performance was noteworthy. This is especially impressive considering that the exported models are entirely Python-based and the tests utilized only one CPU core, despite the machine having 20 cores available.

As mentioned in the previous post, the export feature isn't the final ambition for modelx enhancements. The all-Python nature of exported models opens doors to numerous third-party tools aimed at boosting performance. For instance, exported models could benefit from PyPy, a just-in-time compiler implementation of Python. Moreover, by adding type hints to methods and variables, exported models could transition to statically-typed, compiled models using tools like Cython and mypyc. While automating such transformations might be a challenge, it's a realistic and achievable one. With these enhancements in place, modelx can transition from handling minor tasks, such as model validations, to supporting full-scale production models.