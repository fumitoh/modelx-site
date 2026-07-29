---
layout: post
comments: true
title: "modelx Release Notes Roundup: v0.24.0 to v0.31.1"
categories: modelx
---

It has been a while since the last post here, but modelx development has
continued steadily. This post summarizes the thirteen modelx releases
published since then, from v0.24.0 (December 2023) to v0.31.1 (May 2026),
based on the release notes for each version. For full details, follow the
links to the release notes.

## v0.31.1 (May 31, 2026)

* Added the `info` property for human-readable snapshots of modelx objects.
* Extended the CI test matrix with Python 3.8 and NetworkX 2.5.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_31_1.html)

## v0.31.0 (May 16, 2026)

* Serialized Python files now carry a pseudo-python header (introduced with
  a new serializer version, with backward compatibility restored for
  earlier formats).
* Added macro serialization support.
* Fixes include: an exporter crash on model-level (global) references,
  a crash in `get_node_repr` on unhashable keys when `is_cached=False`,
  a clearer error from `read_model` when the path is not a modelx model,
  and docstring serialization for special characters.
* Added Python 3.14 to the supported classifiers.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_31_0.html)

## v0.30.1 (January 24, 2026)

* Fixed typos in the Mortgage tutorial, contributed by a first-time
  contributor.

[Release page](https://github.com/fumitoh/modelx/releases/tag/v0.30.1)

## v0.30.0 (December 12, 2025)

* Introduced the Macro feature, which lets you define and store Python
  functions as part of a model, saved and loaded together with the model.
* New APIs to support it: the `Macro` class, the `defmacro()` decorator,
  the `new_macro()` method and the `macros` property.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_30_0.html)

## v0.29.2 (December 6, 2025)

* Introduced `new_space_from_model()`, which copies a model into another
  model as a space.
* Fixed an issue where creating child spaces in parameterized parent
  spaces with existing ItemSpaces did not properly clear the ItemSpaces
  and refresh the namespace.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_29_2.html)

## v0.29.1 (November 22, 2025)

* Introduced `export_members()` for exporting space members to a module's
  global namespace.
* Added a tentative `compare_cells()` method for comparing identically
  named cells across spaces.
* Fixed an issue where system references were included in the `refs`
  property.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_29_1.html)

## v0.29.0 (November 2, 2025)

* Major refactoring to improve the core components of modelx.
* `reload()` and `import_funcs()` are deprecated and will be removed in
  future versions.
* Added support for Python 3.14.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_29_0.html)

## v0.28.1 (August 2, 2025)

* Error message tracebacks now include a "Caused by" section.
* Fixed an access-denied error during write operations on Windows.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_28_1.html)

## v0.28.0 (December 8, 2024)

* Reading models with `read_model()` is now significantly faster.
* Improved the error message when accessing non-existent names in a Space.
* Fixed an issue preventing values with unary minus operators from being
  stored as reference variables.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_28_0.html)

## v0.27.0 (August 25, 2024)

* Introduced uncached cells: the `is_cached` property, the `@uncached`
  decorator and `is_cached` parameters on `defcells()` and `new_cells()`
  allow formulas to skip caching, so they can accept unhashable arguments
  such as lists.
* Added `@cached` as an alias of `defcells`.
* Fixed two inheritance issues and extended formula trace messages to
  show up to 20 lines.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_27_0.html)

## v0.26.0 (July 15, 2024)

* Added underscore-prefixed properties (`_parent`, `_name`, `_cells` and
  others) as aliases on Model, Space and Cells objects, for compatibility
  with exported models.
* The `del` statement now works for removing UserSpace parameters and
  ItemSpace objects in exported models.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_26_0.html)

## v0.25.1 (June 13, 2024)

* A bug-fix release addressing
  [GH-127](https://github.com/fumitoh/modelx/issues/127).

[Release page](https://github.com/fumitoh/modelx/releases/tag/v0.25.1)

## v0.25.0 (February 18, 2024)

* Added the `Model.path` property, which lets formulas reference external
  files by paths relative to the model file's location.
* Added support for Python 3.12.
* Backward-incompatible changes: the subscription syntax `[]` for calling
  cells was removed (use `()` instead), along with some Cells methods such
  as `match()`.
* Fixed an export error on Python 3.12 and a `PermissionError` from
  `read_model()` on Windows.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_25_0.html)

## v0.24.0 (December 2, 2023)

* Child spaces are no longer inherited by default when a UserSpace
  inherits from another; this is a backward-incompatible change.
  Dynamic child spaces of parameterized spaces are not affected.
* Removed previously deprecated backup/restore functionality:
  `restore_model()`, `open_model()`, `Model.backup()` and `Model.save()`.

[Release notes](https://docs.modelx.io/en/latest/releases/relnotes_v0_24_0.html)
