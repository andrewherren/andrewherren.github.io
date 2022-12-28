---
title: "scikit-tree"
excerpt: "Cython-based library for rapid experimentation with tree methods"
collection: portfolio
---

[scikit-tree](https://github.com/andrewherren/scikit-tree) extends the Cython implementation of a decision tree learner from `scikit-learn`. 

The general rationale is as follows:

1. `scikit-learn`'s codebase is fast and heavily tested. Relying on their implementation of a tree data structures makes it easier to extend to new methods without worrying about edge cases in the tree code.

2. Cython offers a nice tradeoff between high-level code (readability, maintainability, syntactic sugar, etc...) and low-level code (direct memory access / management, GIL release, etc...)

The initial version of this project is just an extraction of the `scikit-learn` tree code into a minimum-viable python library. As a result, the build system largely follows that of the [scikit-learn-contrib project template](https://github.com/scikit-learn-contrib/project-template) rather than the `scikit-learn` build infrastructure.
