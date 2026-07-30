---
layout: features
title: Why modelx
mermaid: true
redirect_from:
  - /why.html

feature_groups:
    - title: Governance & auditability
      features:
        - title: Dependency tracing
          intro: >
            Dependency tracing is an essential feature for checking and validating
            models. You can check what other values
            each calculated value is using, and also what other values
            it is used by.

        - title: Version control
          intro: >
            modelx models can be saved as text files written in Python syntax,
            which means you can take full advantage of modern version
            control systems, such as Git.

        - title: Document integration
          intro: >
            Document model components, such as spaces and cells
            by their docstrings within their definitions,
            render the docstrings nicely by Sphinx, a documentation
            generator, into html, pdf or other formats.
            No need to maintain a separate model document.

    - title: Model design & productivity
      features:
        - title: Automatic calculation order
          slug: no-programming
          intro: >
            Build models just by writing formulas like you do on
            spreadsheets. No run scripts or orchestration code to
            write&mdash;modelx resolves the execution order from
            formula dependencies.

        - title: Readable formulas
          intro: >
            Define formulas by writing Python functions
            and assign them to Cells. Formulas are evaluated when they are
            called for the first time.
            Lambda expression is also supported.

        - title: Object-oriented
          intro: >
            modelx is object-oriented, which means you create, access or
            make changes to objects, such as models, spaces and cells.
            modelx features composition and inheritance mechanisms
            common in Object-oriented programming (OOP).

        - title: Parameterization
          intro: >
            You can apply the same set of calculations
            to multiple data sets associated with parameters.
            This can be achieved by parameterizing spaces.

        - title: Excel Interface
          intro: >
            Excel files are great for storing data of relatively small sizes.
            You can create new spaces and populate new cells in the space
            with data from Excel files.

        - title: GUI as Spyder plugin
          intro: >
            Spyder is a popular open-source Python IDE.
            Spyder plugin for modelx adds custom IPython consoles
            and GUI widgets to use modelx with Spyder more intuitively.

    - title: Deployment & performance
      prominent: true
      features:
        - title: Export to pure Python
          intro: >
            Export any model as a self-contained Python package with
            no runtime dependency on modelx. Build in modelx,
            deploy as plain Python wherever Python runs.

        - title: Native compilation with modelx-cython
          intro: >
            Exported models can be cythonized and compiled to
            native code with modelx-cython for faster execution.
---

An actuarial modeling system has to deliver four things at once:
**transparency**, so that anyone reviewing the model can see exactly how
each number is produced; **auditability**, so that any result can be traced
back to its inputs; **performance**, so that models run at production
scale; and **maintainability**, so that models keep pace with products,
regulation and staff turnover. modelx approaches all four with a single
principle: actuarial models are code &mdash; plain, readable Python.

This page makes the case by audience, then documents each feature in
detail below.

## Coming from Excel

Spreadsheets made actuaries productive because formulas, not programs, are
the natural unit of actuarial thinking. modelx keeps that: a model is a
collection of formulas, and modelx works out the calculation order for you.
What changes is everything around the formulas:

* **[Dependency tracing](#dependency-tracing) at whole-model scale.**
  Excel's trace-precedents works one cell at a time. In modelx, every
  calculated value knows its precedents and dependents (`preds` /
  `succs`), so you can trace an entire projection from result to
  assumptions.
* **[Real version control.](#version-control)** modelx models are saved
  as plain Python text, so Git gives you meaningful diffs, history,
  branches and code review &mdash; not a folder of dated workbook copies.
* **Testing and automation.** Models are ordinary Python objects: you can
  run them from scripts, write automated tests against them, and put them
  in scheduled jobs or CI pipelines.

Excel is still supported where it is strongest: as a
[data interface](#excel-interface) for reading and writing model inputs
and outputs.

## Coming from plain Python

If you build actuarial models as ordinary Python scripts, much of your code
is not actuarial at all &mdash; it sequences calculations, caches
intermediate results and wires data through functions. modelx removes that
layer:

* **[No orchestration code.](#no-programming)** You declare formulas;
  modelx resolves their execution order from dependencies and caches
  results automatically.
* **Built-in tracing for validation.** Because modelx tracks the
  dependency graph as it calculates, checking a result does not require
  instrumenting your code.
* **Model structure instead of ad-hoc scripts.**
  [Spaces](#object-oriented) group formulas into components; inheritance
  lets products share common logic; [parameterization](#parameterization)
  applies one set of formulas to arbitrary combinations of inputs.
* **[Models double as their own documentation.](#document-integration)**
  Docstrings on models, spaces and cells can be rendered into full model
  documentation with Sphinx.

## Coming from commercial modeling systems

Compared with proprietary modeling platforms, modelx offers a different
set of trade-offs:

* **Every formula is inspectable.** There are no black-box calculation
  engines; the entire model, and modelx itself, can be read and reviewed.
* **No license cost.** modelx is open source under LGPLv3.
* **The open Python ecosystem.** Models work with pandas, NumPy, Git, CI
  services and the rest of the scientific Python stack, and the pool of
  people who can work on them is the pool of people who know Python.
* **Models AI agents can work on.** modelx models are plain text, so AI
  coding agents can read, explain, refactor and test them. See
  [modelx and AI agents](ai.html).
* **No lock-in &mdash; not even to modelx itself.** Models
  [export as self-contained pure-Python packages](#export-to-pure-python)
  that run without modelx, and exported models can be
  [compiled to native code](#native-compilation-with-modelx-cython) with
  modelx-cython. Production performance does not require a proprietary
  runtime.

## Governance & auditability

### Dependency tracing

Dependency tracing is an essential feature for checking and validating models.
You can check what other values each calculated value is using,
and also what other values it is used by.

```py
>>> Balance.preds(5)
[Model1.Space1.Balance(t=4)=400, Model1.Space1.Cashflow(t=5)=100]

>>> Balance.succs(4)
[Model1.Space1.Balance(t=5)=500]
```

Dependency tracing is one of the reasons to choose models-as-code
over spreadsheets and proprietary platforms&mdash;see
[Coming from Excel](#coming-from-excel).

<div class="row">
  <div class="col-sm-6">
    <img src="/img/features/DependencyTracingPrecedents.png" alt="Trace precedents">
  </div>
  <div class="col-sm-6">
    <img src="/img/features/DependencyTracingDependents.png" alt="Trace dependents">
  </div>
</div>

### Version control

modelx models are saved as text files written in the Python syntax, 
which means you can take full 
advantage of modern version control systems and collaborative
software development platforms, such as [Git] and [GitHub].
Plain-text models are also readable by AI coding agents&mdash;see
[modelx and AI agents](ai.html).

[Git]: https://git-scm.com/
[GitHub]: https://github.com/

### Document integration

Documenting models is an integral part of model governance.
modelx enables you to document model components,
such as Models, Spaces and Cells by setting their **doc**
properties. When they are written to files,
they are represented by Python modules and functions,
and their doc propeties are represented by 
the [docstrings] of the Python modules and functions.
Using [Sphinx],
a documentation generator widely used in the Python community
for technical documentation, 
you can generate model documents from the docstrings,
beautifully rendered in HTML, PDF and other formats.

See [lifelib] pages for document samples.
[Sphinx] interpretes [reStructuredText],
a plaintext markup syntax, and
can render math expressions written in LaTex,
images, code samples as well as basic sets
of markups, such as lists and links. 

[docstrings]: https://www.python.org/dev/peps/pep-0257/
[reStructuredText]: https://docutils.sourceforge.io/rst.html
[Sphinx]: https://www.sphinx-doc.org/en/master/
[lifelib]: https://lifelib.io


## Model design & productivity

### Automatic calculation order {#no-programming}

**modelx** enables you to build models
just by defining **Formulas** like you do on spreadsheets.
You can define Formulas by writing Python functions.
modelx automatically resolves the calculation order from 
their dependency, so you don't need to write scripts 
to run your models. 
modelx calculates the results when they are retrieved
for the first time, and the results are kept until
they are cleared or recalculated.

```py
>>> import modelx as mx

>>> @mx.defcells
... def Cashflow(t):
...     return 100

>>> @mx.defcells
... def Balance(t):
...     if t > 0:
...         return Balance(t-1) + Cashflow(t)
...     else:
...         return 0
    
>>> Balance(5)
500

>>> dict(Balance)
{0: 0,
 1: 100,
 2: 200,
 3: 300,
 4: 400,
 5: 500}
```


### Readable formulas

Formulas are defined by [Python functions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions),
so you can enjoy the expressivenss of Python syntax.
Python functions are far more readable, understandable
than spreadsheet formulas.

Python have rich [control flows](https://docs.python.org/3/tutorial/controlflow.html),
and built-in [data structures](https://docs.python.org/3/tutorial/datastructures.html)
such as list and dict. All of these are avaialble for defining Formulas.

Objects are referenced by their names in Formulas.
You can also use [lambda expressions](https://docs.python.org/3/tutorial/controlflow.html#lambda-expressions)
to define simple Formulas.

### Object-oriented

modelx is object-oriented. You create, access or make changes to objects, 
such as **Models**, **Spaces** and **Cells**. 
modelx features [composition] and [inheritance] mechanisms common in Object-oriented programming (OOP).

<div class="row">
  <div class="col-sm-6"  style="text-align:center;">
<pre><code class="language-mermaid">
graph TD
A(Model1) --- B[Space1]
B --- C[Cells1]
B --- D[Space2]
D --- E(Cells2)
</code></pre>
  </div>
<div class="col-sm-6">
<img src="/img/features/ObjectTree.png" alt="Object tree">
</div>
</div>

[composition]: https://en.wikipedia.org/wiki/Object_composition
[inheritance]: https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)

### Parameterization

**Parameterization** is a very powerful feature
to quickly and naturally extend a Space written 
in terms of one combination of input values 
into a parameterized Space accepting arbitrary combinations
of input values as arguments.

Using Space parameterization, you can parameterize
Formula calculations without changing their signatures.


```py
>>> space = mx.new_space("Loan")

>>> @mx.defcells
... def Payment():
...     return 1000 * Rate/100 * (1+Rate/100)**Term / ((1+Rate/100)**Term -1)

>>> space.parameters = ("Rate", "Term")

>>> space[3, 10].Payment()
117.23050660515952

>>> for rate, term in zip((3, 4, 5), (10, 20, 30)):
...     print(space[rate, term].Payment())
117.23050660515952
73.58175032862884
65.05143508027656    
```

![Parameterization](img/features/Parameterization.png)

### Excel Interface

Excel files are great for storing data of relatively small sizes. 
You can create new spaces and populate new cells in the space with data from Excel files.

### GUI as Spyder plugin

Spyder is a popular open-source Python IDE. 
Spyder plugin for modelx adds custom IPython consoles and GUI widgets to use modelx with Spyder more intuitively.


## Deployment & performance

### Export to pure Python

Since modelx 0.22.0, [`Model.export`] lets you export any model as a
self-contained Python package that runs without modelx installed:

```py
>>> model.export("TermLife_ex")

>>> from TermLife_ex import mx_model

>>> mx_model.Projection.net_cf(10)
11.725013975627874
```

The exported package is plain Python code, so it can be deployed
wherever Python runs&mdash;production systems, cloud platforms, or
containers&mdash;without installing modelx.
Some modelx features are not supported by exported models;
see [`Model.export`] in the reference guide for current limitations,
and the blog post
[Introducing the Export Feature]({% post_url 2023-07-29-export-feature-intro %})
for background.

[`Model.export`]: https://docs.modelx.io/en/latest/reference/generated/modelx.core.model.Model.export.html

### Native compilation with modelx-cython

[modelx-cython] translates exported models to [Cython] and compiles
them to native code with a single command:

```console
$ mx2cy TermLife_ex
```

modelx-cython is at an early, experimental stage of development.
See the [modelx-cython] repository for its current status and usage, and the blog post
[Introducing modelx-cython]({% post_url 2023-10-21-introducing-modelx-cython %})
for background and measurements.

[modelx-cython]: https://github.com/fumitoh/modelx-cython
[Cython]: https://cython.org/

## Get started

Install modelx and build your first model in minutes:

```
pip install modelx
# or
conda install -c conda-forge modelx
```

Then follow the
[tutorial](https://docs.modelx.io/en/latest/tutorial/index.html), or
explore working actuarial models at [lifelib](https://lifelib.io).
