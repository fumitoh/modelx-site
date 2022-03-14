---
layout: post
comments: true
title: "Running modelx in parallel using ipyparallel"
categories: lifelib
---

In the post, ["How fast are lifelib models now?"]({% post_url 2022-02-26-how-fast-are-lifelib-models-now %}),
we've seen that lifelib models run pretty fast.
But each of these tests was run by a single process.
In fact, the CPU utilization during any of the test runs was only about 8%.

How can modelx take full advantage of multiple CPU cores?

The answer is to leverage multiprocessing.
You can write a multiprocessing program on your own using 
[multiprocessing](https://docs.python.org/3/library/multiprocessing.html),
the Python standard library for multiprocessing,
which was briefly discussed before.
However, a much easier approach is to use [ipyparallel](https://ipyparallel.readthedocs.io/en/latest/).


## ipyparallel

[ipyparallel](https://ipyparallel.readthedocs.io/en/latest/) lets you run multiple instances of your model in multiple processes on multiple machines.
Here are some features of ipyparallel.

* Easy to handle both synchronous and asynchronous communications.
* Capable of sending and receiving numpy and pandas objects fast to and from engines.
* Capable of communicating with remote machines in the same way as with the localhost.

## Testing ipyparallel

The machine used for the test is the same as the one used for the previous test.

* CPU: 12th Gen Intel Core i7-12700KF (20 logical CPU cores)
* OS: Windows 11
* Memory: 64GB

The relevant results form the previous test are shown below. These results are of single-process runs.

| Model | Python ver. | # Model points | # Steps | # Calcs | Run time (Sec.) |
| ----- | ----------- | -------------- | ------- | ---- | ---- |
| `CashValue_ME` | 3.9.7 | 10K |  1141 | 46820 | 13.61 | 
| `CashValue_ME` | 3.10.2 | 10K |  1141 | 46820 | 9.85 | 

Now we run up to 100K model points by up to 10 processes using ipyparallel.
For example, if 10 processes are run and each process gets 10K model points, and if the parallel runs scale perfectly,
the run time should be as the same as the run with 10K model points by a single process.

The notebook used for the test is available [here](https://github.com/fumitoh/ipyparallel-modelx-demo/blob/main/ipyparalleldemo.ipynb), 
together with the model point samples and a notebook to generate the model points.


## Test results

The table below shows the test summary.
The run time does not include data loading and transfer, and shows only the processing time to calculate `result_pv`.

| ID | Model | Python ver. | # Processes | # Model points  | Run time (Sec.) |
| -- | ----- | ----------- | --------------  | ---- | ---- |
|1| `CashValue_ME` | 3.9.7 | 1| 10K  | 13.61 | 
|2| `CashValue_ME` | 3.9.7 | 1| 100K | 116 | 
|3| `CashValue_ME` | 3.9.7 | 5| 50K | 15 | 
|4| `CashValue_ME` | 3.9.7 | 5| 100K  | 30 | 
|5| `CashValue_ME` | 3.9.7 | 10| 100K  | 22 | 
|6| `CashValue_ME` | 3.10.2 |1 | 10K  | 9.85 | 
|7| `CashValue_ME` | 3.10.2 |1 | 100K | 65 |
|8| `CashValue_ME` | 3.10.2 |5 | 50K  | 11 | 
|9| `CashValue_ME` | 3.10.2 |5 | 100K  | 19 | 
|10| `CashValue_ME` | 3.10.2 |10 | 100K  | 14 | 

<!--- 
| ID | Model | Python ver. | # Processes | # Model points  | Run time (Sec.) |
| -- | ----- | ----------- | --------------  | ---- | ---- |
|1| `CashValue_ME` | 3.9.7 | 1| 10K  | 13.61 | 
|2| `CashValue_ME` | 3.9.7 | 1| 100K | 116 | 
|3| `CashValue_ME` | 3.9.7 | 5| 50K | 37 | 
|4| `CashValue_ME` | 3.9.7 | 5| 100K  | 86 | 
|5| `CashValue_ME` | 3.9.7 | 10| 100K  | 65 | 
|6| `CashValue_ME` | 3.10.2 |1 | 10K  | 9.85 | 
|7| `CashValue_ME` | 3.10.2 |1 | 100K | 65 |
|8| `CashValue_ME` | 3.10.2 |5 | 50K  | 22 | 
|9| `CashValue_ME` | 3.10.2 |5 | 100K  | 47 | 
|10| `CashValue_ME` | 3.10.2 |10 | 100K  | 35 | 
--->


Here findings from the test.

* The runs with 10 processes are 5 times faster than the single process runs.
* Per-process performance gets gradually worse as the number of processes increase. 
* There are a number of formulas that are not proportional to the number of model points.  These formulas should take a fixed amount of time regardless of the number of processes.
* The parallel runs with 10 processes utilize 65% to 70% of the CPU cores while the single process runs utilize only 8%. The 5 process parallel runs utilize 35%.
* Python 3.10 is 25%-45% faster than Python 3.9
* Garbage collection needed be turned off to get the results above. When garbage collection was turned on, the run speed deteriorated significantly.

Note: [Garbage collection](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)) is a mechanism to clean up data that is not referenced by any variable anymore by freeing the memory for the data.


