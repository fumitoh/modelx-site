---
layout: post
comments: true
title: "Testing lifelib on GPU"
categories: lifelib
---

![GPU image](/img/2022-01-08/nvidia-g24c404370_1280.jpg)

[In the previous post]({% post_url 2022-01-08-plan-for-lifelib-with-gpu %}), a plan for running lifelib actuarial models on GPU was presented.
This post is about the initial attempt that has been made to make the plan a reality.

As mentioned previously, [CuPy] is a drop-in replacement for [NumPy], and
[RAPIDS cuDF] is a drop-in replacement for [pandas].
So if everything goes fine, lifelib models should run on GPU with minor modifications
by assigning `cupy` to `np`, and `cudf` to `pd`. 

[NumPy]:https://numpy.org/
[pandas]:https://pandas.pydata.org/
[GPU]:https://en.wikipedia.org/wiki/Graphics_processing_unit
[CuPy]:https://cupy.dev/
[RAPIDS cuDF]:https://rapids.ai/

## Setting Up Test Environment

The machine used for the test has the following specs:

  * OS: Windows 11 
  * CPU: 12th Gen Intel Core i7-12700KF 3.6Ghz, 20 logical cores
  * RAM: 64GB 
  * GPU: Nvidia GeForce 1650, 896 cores
  * GPU RAM: 4GB

It was a disappointment to find out that [RAPIDS cuDF] doesn't support Windows.
Luckily, it's not hard to run Linux on Windows these days.  
Windows is equipped WSL2 ([Windows Subsystem for Linux 2](https://docs.microsoft.com/en-us/windows/wsl/)).
WSL2 lets you install a Linux OS, such as Ubuntu, on a Windows machine,
and use Linux applications as if they are installed on a native Linux machine.  
In old days, this kind of technologies had been available as a virtual machine
by means of software-emulated hardware, so the performance of a guest
OS and its applications was significantly lower than a natively installed
OS and applications.
In contrast,  the guest OS and applications on WSL2 runs as fast as if
they are installed natively, thanks to a native hypervisor technology called 
[Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/).

WSL2 was set up on the Windows host OS, and 
an Anaconda environment was set up in WSL2 for the test.
[CuPy] and [RAPIDS cuDF] were installed without any difficulties.

[Spyder] is capable of connecting to remote IPython kernels,
so you can write Python code in Spyder running on the host Windows and run it on WSL2
as explained [here](https://docs.spyder-ide.org/current/faq.html#install-wsl2).
Unfortunately, [spyder-modelx] does not support remote IPython kernels at the moment.

[Spyder]:https://www.spyder-ide.org/
[spyder-modelx]:https://docs.modelx.io/en/latest/spyder.html

## Making the Model GPU-enabled

The model used for the test was modified from [BasicTerm_ME].
The number of model points in the model point file was increased from 10000 to 1 million. The model points are generated from the same Jupyter notebook in the 
[basiclife] library. The actual model points to run were selected by changing the formula of `model_point()`.
In addition, the file format is changed from Excel to CSV in order to reduce
loading time. The 1 million model points load in no time from the CSV file.
This model is the base model (CPU model) before applying any modifications to make it GPU-enabled,
and was used for measuring run time on CPU for comparison.

As predicted previously, making the GPU-enabled model did not take much effort.
Basically changes are made for either of the two reasons below:

- The data saved in input files, such as model points, mortality and lapse rates,
  are initially loaded as pandas Series or DataFrames, so additional code is needed to convert the loaded objects to equivalent cuDF objects.

- Some methods and parameters existing in NumPy/pandas are missing in CuPy/cuDF,
  so formulas containing such methods or parameters needed modifications.
  
The complete comparison of the formulas is available [here](https://www.diffchecker.com/LzyRRKxa).
The models, both the CPU and GPU models used for the test are archived in [this file]({{site.url}}/download/2022-01-15/BasicTerm_ME_TestOnGPU.zip)
(modelx v0.18.0 or newer is required to run the models).
It is possible to make the same model run both on CPU and GPU,
but to achieve that, modelx needs further enhancement. 

## Test Results

The CPU model was run on the host Windows, and on the guest Linux (WSL2) for comparison,
and the GPU model was run on WSL2 with various sizes of model points. 
All runs have 227 projection steps.

| Platform | Proc. | # Model points | Run time (Sec.) | Sec. per 10K MPs | Util.% | Max RAM Usage |
| -------- | ---- | ---- | ---- | ---- | ---- | ---- |
| Host Win. | CPU | 50K | 5.5 | 1 | 8% | 11GB |
| Host Win. | CPU | 500K | 53 | 1 |8% | 28GB |
| Host Win. | CPU | 1M |  106 | 1 | 8% | 43GB |
| WSL2 | CPU | 50K |  5 | 1 | 9% | 11GB |
| WSL2 | CPU | 500K | 43 | 0.9 | 9% | 28GB |
| WSL2 | GPU | 50K |  12 | 2 | 97% | 3.7GB |

The processor utilization % and max RAM usage in the table were figures
observed in Windows task manger, so usage of other applications are included.

### Memory Usage Observation

The host Windows has 64GB memory and the run with 1 million model points consumed about 40GB of it. Initially, the host Windows only had 32GB memory and later additional 32GB memory was physically installed, 
but even before the expansion to 64GB, it was capable of running the entire 1 million model points in almost within the same time, thanks to Windows' memory compression feature.

A run with the 1 million model points was also tested on WSL2, which gets 32GB memory (50% of the host). In contrast to the host Windows, WSL2 terminated
the run around when the memory consumption reached the limit and no memory compression kicked in.

### Performance Observation

Before discussing the GPU results, it is worth mentioning that the runs on WSL2 performed
better than the runs on the host Windows.
It is not certain as to why the WSL2 runs are faster, may be it's because of the difference
of the versions of Python and its libraries, or because of the OS difference,
but this proves how good the Hyper-V technology allows WSL2 to perform compute-intensive tasks 
as mentioned above. 

As the table shows, CPU performed much better than GPU. Looking at the results of the
directly comparable runs with 50K model points, CPU is about twice as fast as GPU, despite the fact that 
CPU only utilizes 8-9% of its entire 20 cores, while GPU utilizes 97% of its entire 896 cores.

Here are some possible causes for the GPU's poor result:

* This may simply due to the GeForce being not primarily meant for scientific computing. 
  The GeForce product line is for gaming, targeting individual gamers.
  The double-precision performance of GeForce 1650 is about
  [93.24 GFLOPS](https://www.techpowerup.com/gpu-specs/geforce-gtx-1650.c3366),
  while [V100](https://images.nvidia.com/content/technologies/volta/pdf/tesla-volta-v100-datasheet-letter-fnl-web.pdf) marks 7-7.8 TFLOPS.

* Some vector and matrix operations, such as merge and concatenation, may not be optimal for GPU.

* Another possible cause is the overhead required for loading data
  onto GPU RAM. The GPU-enabled model loads data on RAM as pandas objects initially, then 
  converts it to cuDF objects during an early stage of the runs, at which point
  the contained data is copied onto the GPU RAM. This process is additional
  to the logic to the original CPU model, and can be eliminated once modelx
  supports loading data directly from files as cuDF objects. 

## Next Step

Obviously, it is too early to conclude that CPU performs better than GPU in running lifelib actuarial models. At least, following steps should give more insights.

- Inspect the stack trace of the runs and see what formulas are taking time.
- Run the models on an GPU-enabled cloud environment, namely AWS EC2 P3.

## Issues

Through this test one critical issue has been observed, which is the excessive memory consumption of lifelib models.
The massive memory consumption limits the number of model points that can be processed at once.
This is more concern for GPU models because the size of GPU RAM is usually smaller than CPU RAM. 

Reducing the number of projection steps should work as a quick fix. 
Another more fundamental approach to solve the memory issue is to implement a "low-memory" production mode in modelx.

##  Future Development Plan

In addition to addressing the issue raised above, following enhancements should help modelx
more GPU friendly.

* The `new_pandas` should support reading data from files directly cuDF objects,
  which may reduce the overhead mentioned above.

* spyder-modelx currently does not support remote IPython session. 
  It'd be nice if the modelx plugin on a host machine works with IPython sessions
  running remotely, say, on WSL2.



[lifelib]:https://lifelib.io/
[basiclife]:https://lifelib.io/libraries/basiclife/index.html
[savings]:https://lifelib.io/libraries/savings/index.html

[BasicTerm_M]:https://lifelib.io/libraries/basiclife/BasicTerm_M.html
[BasicTerm_ME]:https://lifelib.io/libraries/basiclife/BasicTerm_ME.html
[CashValue_ME]:https://lifelib.io/libraries/savings/CashValue_ME.html


