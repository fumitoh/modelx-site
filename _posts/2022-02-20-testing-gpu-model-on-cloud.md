---
layout: post
comments: true
title: "Testing GPU model on Cloud"
categories: lifelib
---

Recently a test to run a lifelib model on GPU has been conducted as detailed 
[in this post]({% post_url 2022-01-15-testing-lifelib-on-gpu %}).
The GPU used for that test was Nvidia GeForce 1650 on a consumer desktop PC, which is primarily for gaming. The next step was to conduct the same test using GPU on a cloud platform,
which is much more powerful than GeForce.

Finally, the test was performed using a high-performance GPU for accelerated
computing available on Google Cloud Platform (GCP). This post is about how the test went.


## Profiling the GPU model

Before running the test model on GCP, the model was profiled.
The table below shows the top 3 of the most time-consuming formulas.

| Formulas         | No. of calls | Duration (secs.) |
| ---------------- | ------------ | ---------------- |
| `max_proj_len()` | 1            | 3.857875109      |
| `mort_rate(t)`   | 277          | 2.302189112      |
| `model_point()`  | 1            | 0.788285255      |

The profiling result showed that the `max_proj_len` formula 
was taking nearly 4 seconds of the total run time of 10 seconds,
despite that it was called only once.
The `max_proj_len` formula was defined as below:

```python
lambda: int(max(proj_len()))
```

`proj_len()` returns a [CuPy] array, each element of which holds the length of the projection steps of each model point. It seems that the `max` standard function operates on the [CuPy] array
inefficiently.
Based on the analysis, the formula of `max_proj_len` was changed as below:

```python
def max_proj_len():
    return int(proj_len().max())
```

After the change `max_proj_len` took almost no time and the total run time was
reduced from 10 to 6 seconds.


[CuPy]: https://cupy.dev/

## Cloud GPU shortage

Due to high demand for GPU accelerated computing capacity and a severe shortage of GPU supply, among GCP (Google Cloud Platform), AWS and Azure, 
GCP was the only provider that approved a quota to use a GPU enabled machine.
But even on GCP, it was not always possible to launch a GPU enabled-machine in January, due to resource exhaustion.


## Running the model on Google Cloud Platform

Here are the specs of the GCP virtual machine(VM) instance used for the test. 

- Machine type: n1-standard-8
- GPU: Nvidia Tesla V100
- GPU memory size: 16 GB
- OS: Ubuntu 20.04

Jupyter notebook server was run on the VM, and the model was uploaded and run on 
the VM through Jupyter notebook in the browser running on the local PC.
The actual notebook used for the test is [here]({{site.url}}/download/2022-02-20/Run_GPU_trace.ipynb).

The model ran with 250,000 model points successfully, and it took about 12 seconds, as detailed in this [stack trace]({{site.url}}/download/2022-02-20/stacktrace.xlsx).
The model failed to run with 300,000 due to GPU memory exhaustion.

Since the same model with 50,000 model points took 6 seconds on the local GPU,
The models runs only 2.5 time faster on the cloud GPU than on local GPU.
During the run on the cloud GPU, GPU utilization was monitored by [`gpustat`](https://github.com/wookayin/gpustat), and
the utilization % was up to only 20% at maximum. 


Although the test result is not very impressive, it does not mean actuarial models runs slow on GPU. GPU cores have potential to make actuarial models run drastically fast
when they are fully utilized.
However, to make a model run effectively on GPU, 
the model needs to be optimized delicately. In the case of this test, one small tweak saved 40% of the total run time.
Also, the cost to transferring data to and from GPU memory needs
to be taken into consideration and minimized.  
