---
layout: page
title: modelx and AI agents
---

AI coding agents &mdash; Claude Code and tools like it &mdash; operate on
plain-text code. That is exactly what a modelx model is. Models are saved
as plain Python text, so an agent can read an entire model the same way it
reads any codebase: explain what a projection does, trace how an assumption
flows through it, refactor a space, extend a product, or write tests for
the results. Nothing about the model is opaque to the agent, because
nothing about the model is opaque to anyone.

The same applies after deployment. modelx models export as self-contained
pure-Python packages, and those packages are standard Python code:
AI agents and CI pipelines can import them, run them and test them
directly, with no modelx-specific tooling in the loop.

## Plain text is the interface

This is a practical difference, stated factually: proprietary binary model
formats are not directly readable by coding agents. An agent can only work
on what it can read, and what it can read is text. Actuarial models built
as code get the full benefit of the current generation of AI tooling
&mdash; and of whatever comes next, since plain text is the one format
every generation of developer tooling has supported.

Concretely, in the modelx ecosystem:

* lifelib templates can ship `CLAUDE.md` project guidance so that agents
  start with the right context about a model's structure and conventions.
* A modelx actuarial skill exists for Claude, packaging modelx know-how
  for agent use.

<!-- TODO(fumito): names/links for the skill and CLAUDE.md examples -->

## Governance for agent-assisted modeling

For a governance-minded audience, the right question about AI agents is
not whether to allow them, but how to review their output. With modelx,
the answer is the same as for any model change, because agent output *is*
a model change:

* **Git diffs** show exactly what the agent changed, line by line, and
  changes go through the same review that any human change would.
* **Dependency tracing** shows how a changed formula affects results,
  from any value back to its precedents.
* **Tests** run against the model &mdash; or against its exported
  pure-Python package &mdash; catch regressions automatically.

These guardrails exist precisely because the models are code. A model
that lives in a binary format can neither be edited by an agent nor
reviewed with diffs; a model that lives in plain text can be both. That
is the argument for models-as-code: the same properties that make models
auditable by humans make them safely workable by agents.

## Get started

```
pip install modelx
# or
conda install -c conda-forge modelx
```

Read [Why modelx](features.html) for the full case, or start with the
[tutorial](https://docs.modelx.io/en/latest/tutorial/index.html).
