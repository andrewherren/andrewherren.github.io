---
title: 'Experimenting with tree models using the scikit-learn codebase'
date: 2022-12-28
permalink: /posts/2022/12/scikit-tree-blog/
tags:
  - tree models
  - cython
  - scikit-learn
---

**TL;DR**: [scikit-tree](https://github.com/andrewherren/scikit-tree) extends the scikit-learn tree codebase to enable fast, cython-accelerated experimentation with tree algorithms.

**The longer story**: 
Tree-based models are extremely popular for working with tabular data. 
They build on the conceptually simple idea of 

1. Partitioning a dataset
2. Making different predictions on each partition

This makes (small, individual) trees easy to visualize and easy to explain to stakeholders, relative to other machine learning methods. A common use case for trees is to combine the predictions of many small trees into an "ensemble" (gradient boosted decision trees, random forests, etc...).

While there are many open source projects that offer high-quality tree / tree ensemble algorithms (to name a few, 
[scikit-learn](https://scikit-learn.org/stable/), [xgboost](https://xgboost.readthedocs.io/en/stable/), and [lightgbm](https://lightgbm.readthedocs.io/en/latest/)), the R / Python implementations are designed to *run* a specific algorithm or set of algorithms, not to expose a general-purpose API that allows for custom implementations of decision tree algorithms.

This means that, in most cases, if you have a *truly unique* idea that involves decision trees and you want to implement and test the idea, your options are:

* Prototype it (including the tree data structure) in a high-level language, such as R or Python
* Implement it (or at least the tree subset of your project) in a low-level language, such as C or C++

The first option means your code and algorithm will be considerably slower than xgboost, lightgbm, or scikit-learn. (Largely because R and Python are slow for the sorts of iterative, memory-accessing operations that tree models need to do.) So you'll develop and test your algorithm while it's slow, knowing that if it's a good idea, you have to rewrite a lot of the code.

The second option means that you have to write a lot of new C / C++ code, so you'll spend much more of your research time debugging memory leaks and segfaults than you'd probably prefer.

Before you object, "what about [X language]," yes, I am sure there are languages that don't present as extreme a tradeoff between program speed and development time. I'm writing from the perspective of machine learning tools, where the front end typically needs to be R or Python to reach a broad swath of users. And yes, it is generally possible to wrap R / Python around fast code from other languages (Julia, Rust, etc...). I don't speak for every use case here, just my observations about popular libraries for tree methods.

(*Side note: I am not dismissing use cases that involve Julia / Rust / etc, I'm actually rather curious about them! If you've been on a journey where you thought about this problem and decided to work with a language other than R / Python / C / C++, I'd be very interested to hear about that!*)

So ... is this post just me being bummed out about coding decision trees? No! I want to talk about the "path of least resistance" that I've found for implementing and developing reasonably-performant tree methods in python. Rather than write a bunch of custom C / C++ code, I extracted all of (and only) the necessary components of the scikit-learn cython codebase needed to build decision trees and tree ensembles. If you want to get a sense of what this sklearn codebase looks like, it's [here](https://github.com/scikit-learn/scikit-learn/tree/main/sklearn/tree). 

## First of all, what is cython??

[Cython](https://cython.org) is a superset of the python language that can be compiled into C / C++. Its main advantage is that the syntax resembles standard python but also supports types, memory management, C / C++ libraries, releasing the GIL, etc... In short, it's less verbose to read and write than standard C / C++ and faster than standard python.

Cython is used heavily in scikit-learn and also in scipy, numpy and other scientific python libraries. The tree data structure implementation looks like [this](https://github.com/scikit-learn/scikit-learn/blob/main/sklearn/tree/_tree.pyx#L511).

## Sounds great! Where to go from here?

Well, one option is just to fork the entire scikit-learn project, and make any changes you want to make to the tree code. This has a few downsides that most involve working a codebase that implements hundreds of other methods:

1. You have the build all of scikit-learn in order to build your project. (Though after the first build, you should be able to take advantage of caching, etc...)
2. If your idea is good and you want to put it out there open source, you'd distribute a very strange combination of unmodified sklearn estimators and then your new tree method. That means your project will be heavy and confusing to users.

As an alternative, we can follow the example of the folks at [EconML](https://github.com/microsoft/EconML), a Microsoft library for causal inference that implements Honest Trees and Generalized Random Forests (GRF) using a modified version of the scikit-learn cython tree code.

Since cython has to be compiled and then linked to python as an extension module, the easiest way to do this is to just start a new python package and let `setuptools` handle the build.

Since we're going to be extending the scikit-learn code, we can use a relatively simple template for writing packages with scikit-learn-compatible APIs (i.e. classes with `.fit()`, `.predict()`, etc... methods): the [scikit-learn-contrib project template](https://github.com/scikit-learn-contrib/project-template).

## Creating a new python project

The `scikit-learn-contrib` documentation provides a command-line-based approach. I didn't personally want to use everything from that `scikit-learn-contrib` template, so I copied over the bare minimum needed to create python library with a structure that mirrors the scikit-learn project.

### 1. Top-level repo files

At the top level of the repo sit many important build, documentation, and config files. 

The following two files are more or less the defaults created by Github:

* `LICENSE`: scikit-learn uses the BSD-3 license, so I mirrored that in this repository. Other BSD-compatible licenses should also work here.
* `.gitignore`: the standard python `.gitignore` from Github works here, although I added `.c` and `.cpp` since cython creates these source files as byproducts of the build process.

Build files **could** be taken to mirror the build system of scikit-learn, but (a) scikit-learn is a large, complex project and we are extracting and building on a small subset of its codebase and (b) scikit-learn uses numpy distutils, which is in the process of being deprecated in favor of `setuptools`. Here, it's best to start with the build system of `scikit-learn-contrib` and make several modifications:

* `setup.py`: Fill out relevant project info in the `DISTNAME = ...`, `DESCRIPTION = ...` fields. We'll modify this file later once we've added cython code to the project. To start, I removed the comment `"""A template for scikit-learn compatible packages."""` as well as the classifier `'Programming Language :: Python :: 2.7',`. I also added `, 'cython'` to the `INSTALL_REQUIRES` list.
* `setup.cfg`: Can be recycled from `scikit-learn-contrib`

This file can be written from scratch, since this project is brand new and doesn't need much documentation yet:

* `README.rst`: Written as below

```
.. -*- mode: rst -*-

**scikit-tree** is a project designed to make 
experimentation with tree-based machine learning 
methods straightforward. It relies on, and 
extends, the tree code in `scikit-learn`.
```

### 2. Library subfolder

Create a new folder in the repo called `skltree`. Before writing any new code, we will simply set this project up to implement scikit-learn's decision tree classes, so we need to create a file called `__init__.py` with the following code:

```
from .tree._classes import BaseDecisionTree
from .tree._classes import DecisionTreeClassifier
from .tree._classes import DecisionTreeRegressor
from .tree._classes import ExtraTreeClassifier
from .tree._classes import ExtraTreeRegressor

from ._version import __version__

__all__ = [
    "BaseDecisionTree",
    "DecisionTreeClassifier",
    "DecisionTreeRegressor",
    "ExtraTreeClassifier",
    "ExtraTreeRegressor",
]
```

The scikit-learn-contrib repo reads a version number from the file `_version.py` so we initialize this file with

```
__version__ = "0.0.1"
```

Now, within `skltree`, we create a subfolder `tree` as in `sklearn` and include the following files from scikit-learn:

* `__init__.py`
* `_classes.py`
* `_criterion.pxd`
* `_criterion.pyx`
* `_export.py`
* `_reingold_tilford.py`
* `_splitter.pxd`
* `_splitter.pyx`
* `_tree.pxd`
* `_tree_.pyx`
* `_utils.pxd`
* `_utils_.pyx`

Now, some of the above files, when pulled directly from the `scikit-learn` codebase, depend on other `scikit-learn` modules. Some of these dependencies are based on cython code that is only exposed internally within `scikit-learn` at build time, so it is not possible to replace every import with `from sklearn.[module] import [function]`.

The simplest fix is to vendor the relevant cython code from other sklearn modules with this library. To achieve this, the following modifications are made:

#### **Global modifications**

#### `_random.pxd`

Add this file from scikit-learn's `sklearn/utils` submodule

#### `_random.pyx`

Add this file from scikit-learn's `sklearn/utils` submodule and change 

```
from . import check_random_state
```

to 

```
from sklearn.utils.validation import check_random_state
```

#### `_utils.pxd`

Add the following code:

```
# From sklearn.neighbors._quad_tree
# XXX: Careful to not change the order of the arguments. It is important to
# have is_leaf and max_width consecutive as it permits to avoid padding by
# the compiler and keep the size coherent for both C and numpy data structures.
cdef struct Cell:
    # Base storage structure for cells in a QuadTree object

    # Tree structure
    SIZE_t parent              # Parent cell of this cell
    SIZE_t[8] children         # Array pointing to children of this cell

    # Cell description
    SIZE_t cell_id             # Id of the cell in the cells array in the Tree
    SIZE_t point_index         # Index of the point at this cell (only defined
                               # in non empty leaf)
    bint is_leaf               # Does this cell have children?
    DTYPE_t squared_max_width  # Squared value of the maximum width w
    SIZE_t depth               # Depth of the cell in the tree
    SIZE_t cumulative_size     # Number of points included in the subtree with
                               # this cell as a root.

    # Internal constants
    DTYPE_t[3] center          # Store the center for quick split of cells
    DTYPE_t[3] barycenter      # Keep track of the center of mass of the cell

    # Cell boundaries
    DTYPE_t[3] min_bounds      # Inferior boundaries of this cell (inclusive)
    DTYPE_t[3] max_bounds      # Superior boundaries of this cell (exclusive)
```

and remove

```
from ..neighbors._quad_tree cimport Cell
```

#### `_utils.pyx`

Add the following code

```
# From sklearn.neighbors._quad_tree
# Build the corresponding numpy dtype for Cell.
# This works by casting `dummy` to an array of Cell of length 1, which numpy
# can construct a `dtype`-object for. See https://stackoverflow.com/q/62448946
# for a more detailed explanation.
cdef Cell dummy;
CELL_DTYPE = np.asarray(<Cell[:1]>(&dummy)).dtype

assert CELL_DTYPE.itemsize == sizeof(Cell)
```

Then add 

```
import numpy as np
```

to the imports at beginning of the file and change 

```
from ..utils._random cimport our_rand_r
```

to 

```
from ._random cimport our_rand_r
```

#### `_classes.py`

Change any references to `from ..[sklearn library name] import ...` to `from sklearn.[sklearn library name] import ...`. The only exception, which will be made clear below, is

```
from ..utils._param_validation import Hidden, Interval, StrOptions
```

#### `_export.py`

Change any references to `from ..[sklearn library name] import ...` to `from sklearn.[sklearn library name] import ...`

#### `_param_validation.py`

Some of the validation code used to create scikit-learn estimators depends on this file in the scikit-learn project which is not entirely included in the public scikit-learn API. The easiest way to solve this and keep moving is to vendor this file in a `skltree/utils` subfolder. Just change `from .validation import _is_arraylike_not_scalar` to `from sklearn.utils.validation import _is_arraylike_not_scalar`.

### 3. Library tests subfolder

Finally, we port the unit tests from the scikit-learn project, so that as we make changes to the codebase, we can ensure the original tree code still works as intended.

Create a subfolder within `_tree` called `tests` and place the following files from scikit-learn:

* `__init__.py`
* `test_export.py`
* `test_reingold_tilford.py`
* `test_tree.py`

and change any imports that reference `sklearn.tree` to reference `skltree.tree`

### 4. `setup.py` file

We need to add the cython extension modules to the default `setup.py` included in the `scikit-learn-contrib` project.

```
libraries = []
if os.name == "posix":
    libraries.append("m")

cython_extensions = [
    Extension(name = "skltree.tree._tree", sources = ["skltree/tree/_tree.pyx"], include_dirs = [np.get_include()], libraries = libraries, language = "c++", extra_compile_args = ["-O3"]), 
    Extension(name = "skltree.tree._splitter", sources = ["skltree/tree/_splitter.pyx"], include_dirs = [np.get_include()], libraries = libraries, extra_compile_args = ["-O3"]), 
    Extension(name = "skltree.tree._criterion", sources = ["skltree/tree/_criterion.pyx"], include_dirs = [np.get_include()], libraries = libraries, extra_compile_args = ["-O3"]), 
    Extension(name = "skltree.tree._utils", sources = ["skltree/tree/_utils.pyx"], include_dirs = [np.get_include()], libraries = libraries, extra_compile_args = ["-O3"]),
    Extension(name = "skltree.tree._random", sources = ["skltree/tree/_random.pyx"], include_dirs = [np.get_include()], libraries = libraries, extra_compile_args = ["-O3"]), 
]
```

in the setup command, we add a line

```
ext_modules=cythonize(cython_extensions, compiler_directives={
    "language_level": 3,
    "boundscheck": False,
    "wraparound": False,
    "initializedcheck": False,
    "nonecheck": False,
    "cdivision": True,
}), 
```

and in the import lines, we add

```
import numpy as np
from setuptools.extension import Extension
from Cython.Build import cythonize
```

### 5. Example code

Finally, we need one or several minimal examples so that we can run the estimators after they have been built and made available as a package. To start, simply include `scikit-learn/examples/tree/plot_regression_tree.py` in this project as `scikit-tree/examples/tree/plot_regression_tree.py` and change the import statement to reference `skltree.tree`.

We will add some more minimal runner scripts later once the project has been built.

## Building the project

First, we need to set up a python environment with necessary dependencies to build this code. Here, we largely mirror the scikit-learn developer guidelines. For mac and linux users, this can be done via conda with

```
conda create -n partition_env -c conda-forge python=3.10 numpy scipy cython pytest matplotlib pandas \
    joblib threadpoolctl pytest compilers llvm-openmp
```

We activate the environment with

```
conda activate partition_env
```

And build the library by navigating to the local copy of the repo and installing the package locally via pip

```
cd ~/[path to folder]/scikit-tree
python setup.py clean
pip install --no-build-isolation -e .
```

This may take some time but if it completes successfully, we are ready for the next step by running the unveil_tree example. In the same terminal, run

```
python -m examples.tree.plot_regression_tree
```

Now, we have a fast, cython-backed codebase that we can use to implement various tree algorithms!
