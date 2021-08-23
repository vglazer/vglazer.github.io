---
layout: post
title:  "Calling C++ from Python"
date:   2015-07-18 17:05:19
permalink: calling-python-from-cpp
categories: programming python C++ bindings SWIG Boost Cython
---
# The Problem

Say you've written a library in C++ and you want to call it from Python. You basically have three options:

* [SWIG](http://swig.org)
* [Boost.Python](http://www.boost.org/doc/libs/1_58_0/libs/python/doc/)
* [Cython](http://cython.org/)

Below I try to give you a flavor for each.

## SWIG

Given an *interface file* specifying the headers you wish to wrap plus some 
wrapping directives, SWIG will generate a single source file containing an 
`extern "C"` API for all relevant classes. It will also generate a Python 
module (e.g. `mylib.py`) that creates an object-oriented overlay on top of 
the C API. In this approach the Python API will largely mirror the C++ API, 
though you do have the ability to add new methods as part of the wrapping 
process. Depending on the complexity of the code you are wrapping you may 
need to fiddle with the interface file, for example to instantiate templates 
and catch exceptions.

Next, you compile the wrapper file and link the resulting object file together
with the library being wrapped as well as any external dependencies into a 
single shared library named `_mylib.so`. In order for this to work you must 
add Python headers such as `pyconfig.h` to your include path and the python 
interpreter (e.g. `libpython2.7`) to your library path. You will also need 
to make sure all object code is position-independent, which might mean 
recompiling externals with `-fPIC`.

## Boost.Python

Instead of generating the wrappers for you based on an *interface file*, 
Boost.Python effectively gives you a template-based DSL for manually 
specifying how to map C++ classes onto Python classes. Although this provides 
very fine-grained control over the Python API, allowing you to e.g. turn 
getters into properties, it does mean that each class has to be wrapped 
individually.

Once you are done crafting the wrapper code you compile and link it the same 
as with SWIG, except that in this case there is no `*.py` file and you directly 
import `mylib.so` in python.

## Cython

Cython is similar to SWIG in that it generates wrappers for you, but instead of 
a single *interface file* you have a `*.pxd` file and a `*.pyx` file which are 
both written in Cython (a kind of extended Python). The 
[examples](http://docs.cython.org/src/userguide/wrapping_CPlusPlus.html) 
I [found](http://blog.perrygeo.net/2008/04/19/a-quick-cython-introduction/) 
[online](https://github.com/cython/cython/wiki/WrappingSetOfCppClasses) tended 
to leave various details out, so I put together a working toy example, found 
below. It assumes that [Anaconda](http://continuum.io/downloads) is installed in 
`$ANACONDA_DIR` and that you `touch include/wrappers.hpp`.

`include/Adder.hpp`:
{% gist vglazer/cb65d6e46a3d04f0cb8c %}

`src/Adder.cpp`:
{% gist vglazer/476dfc4826b70f319be7 %}

`include/Multiplier.hpp`:
{% gist vglazer/513267382190d7479031 %}

`src/Multiplier.cpp`:
{% gist vglazer/18e01d57b59579379f43 %}

`wrappers.pxd`:
{% gist vglazer/2ebedc25d349bf3e775f %}

`wrappers.pyx`:
{% gist vglazer/0e2da818c2fd1cbe573f %}

`Makefile`:
{% gist vglazer/7602aada0da375e99557 %}
