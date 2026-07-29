---
layout: page
title: Why modelx
---

An actuarial modeling system has to deliver four things at once:
**transparency**, so that anyone reviewing the model can see exactly how
each number is produced; **auditability**, so that any result can be traced
back to its inputs; **performance**, so that models run at production
scale; and **maintainability**, so that models keep pace with products,
regulation and staff turnover. modelx approaches all four with a single
principle: actuarial models are code &mdash; plain, readable Python.

## Coming from Excel

Spreadsheets made actuaries productive because formulas, not programs, are
the natural unit of actuarial thinking. modelx keeps that: a model is a
collection of formulas, and modelx works out the calculation order for you.
What changes is everything around the formulas:

* **Dependency tracing at whole-model scale.** Excel's trace-precedents
  works one cell at a time. In modelx, every calculated value knows its
  precedents and dependents (`preds` / `succs`), so you can trace an entire
  projection from result to assumptions.
* **Real version control.** modelx models are saved as plain Python text,
  so Git gives you meaningful diffs, history, branches and code review
  &mdash; not a folder of dated workbook copies.
* **Testing and automation.** Models are ordinary Python objects: you can
  run them from scripts, write automated tests against them, and put them
  in scheduled jobs or CI pipelines.

Excel is still supported where it is strongest: as a data interface for
reading and writing model inputs and outputs.

## Coming from plain Python

If you build actuarial models as ordinary Python scripts, much of your code
is not actuarial at all &mdash; it sequences calculations, caches
intermediate results and wires data through functions. modelx removes that
layer:

* **No orchestration code.** You declare formulas; modelx resolves their
  execution order from dependencies and caches results automatically.
* **Built-in tracing for validation.** Because modelx tracks the
  dependency graph as it calculates, checking a result does not require
  instrumenting your code.
* **Model structure instead of ad-hoc scripts.** Spaces group formulas
  into components; inheritance lets products share common logic;
  parameterization applies one set of formulas to arbitrary combinations
  of inputs.
* **Models double as their own documentation.** Docstrings on models,
  spaces and cells can be rendered into full model documentation with
  Sphinx.

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
* **No lock-in &mdash; not even to modelx itself.** Models export as
  self-contained pure-Python packages that run without modelx, and
  exported models can be compiled to native code with
  [modelx-cython](https://github.com/fumitoh/modelx-cython). Production
  performance does not require a proprietary runtime.

## Honest tradeoffs

modelx is not a commercial product, and it is fair to be clear about what
that means:

* **No vendor support contract.** Support is community-based, primarily
  through [GitHub Discussions](https://github.com/fumitoh/modelx/discussions).
* **You build your own production models.**
  [lifelib](https://lifelib.io) provides working examples and reference
  implementations to start from, not turnkey production models.
* **A smaller ecosystem.** The community of consultants, tools and
  training around modelx is smaller than those of long-established
  commercial vendors.

For teams that want full control over their models &mdash; and the ability
to read every line of them &mdash; these trade-offs are usually the point.

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
