#The Problem
Say you've written a library in C++ and you want to call it from Python. You basically have three options:

* [SWIG](http://swig.org)
* [Boost.Python](http://www.boost.org/doc/libs/1_58_0/libs/python/doc/)
* [Cython](http://cython.org/)

##SWIG
Given an *interface file* specifying the headers you wish to wrap plus some wrapping directives, SWIG will generate a single source file containing an `extern "C"` API for all relevant classes. It will also generate a Python module (e.g. `mylib.py`) that creates an object-oriented overlay on top of the C API. In this approach the Python API will largely mirror the C++ API, though you do have the ability to add new methods as part of the wrapping process. Depending on the complexity of the code you are wrapping you may need to fiddle with the interface file, for example to instantiate templates and catch exceptions.

Next, you compile the wrapper file and link the resulting object file together with the library being wrapped as well as any external dependencies into a single shared library named `_mylib.so`. In order for this to work you must add Python headers such as `pyconfig.h` to your include path and the python interpreter (e.g. `libpython2.7`) to your library path. You will also need to make sure all object code is position-independent, which might mean recompiling externals with `-fPIC`.

##Boost.Python
Instead of generating the wrappers for you based on an *interface file*, Boost.Python effectively gives you a template-based DSL for manually specifying how to map C++ classes onto Python classes. Although this provides very fine-grained control over the Python API, allowing you to e.g. turn getters into properties, it does mean that each class has to be wrapped individually.

Once you are done crafting the wrapper code you compile and link it the same as with SWIG, except that in this case there is no `*.py` file and you directly import `mylib.so` in python.

##Cython
Cython is similar to SWIG in that it generates wrappers for you, but instead of a single *interface file* you have a `*.pxd` file and a `*.pyx` file which are both written in Cython (a kind of extended Python). The [examples](http://docs.cython.org/src/userguide/wrapping_CPlusPlus.html) I [found](http://blog.perrygeo.net/2008/04/19/a-quick-cython-introduction/) [online](https://github.com/cython/cython/wiki/WrappingSetOfCppClasses) tended to leave some details out, so I put together a complete, working example, found below. This assumes Anaconda is installed in `$ANACONDA_DIR` and that you `touch include/wrappers.hpp`.

~~~ cpp
include/Adder.hpp:

#if !defined(__ADDER_HPP__)
#define __ADDER_HPP__

class Adder
{
    public:
            Adder();
            void add(int n);
            int get_sum() const;

    private:
            int _sum;
};

#endif // __ADDER_HPP__
~~~

~~~cpp
src/Adder.cpp:

#include <Adder.hpp>

Adder::Adder()
  : _sum(0)
{}

void 
Adder::add(int n)
{
    _sum += n;
}

int 
Adder::get_sum() const
{
    return _sum;
}
~~~

~~~cpp
include/Multiplier.hpp:

#if !defined(__MULTIPLIER_HPP__)
#define __MULTIPLIER_HPP__

class Multiplier
{
    public:
            Multiplier();
            void mult(int n);
            int get_product() const;

    private:
            int _product;
};

#endif // __MULTIPLIER_HPP__
~~~

~~~cpp
src/Multiplier.cpp:

#include <Multiplier.hpp>

Multiplier::Multiplier()
  : _product(1)
{}

void 
Multiplier::mult(int n)
{
    _product *= n;
}

int 
Multiplier::get_product() const
{
    return _product;
}
~~~

~~~python
wrappers.pxd:

cdef extern from "Adder.hpp":
    cdef cppclass Adder:
        Adder()
        void add(int n)
        int get_sum() const

cdef extern from "Multiplier.hpp":
    cdef cppclass Multiplier:
        Multiplier()
        void mult(int n)
        int get_product() const
~~~

~~~python
wrappers.pyx:

cimport wrappers

cdef class PyAdder:
    cdef wrappers.Adder *thisptr

    def __cinit__(self):
        self.thisptr = new wrappers.Adder()

    def __dealloc__(self):
        del self.thisptr

    def add(self, n):
        self.thisptr.add(n)

    property sum:
        def __get__(self):
            return self.thisptr.get_sum()

cdef class PyMultiplier:
    cdef wrappers.Multiplier *thisptr

    def __cinit__(self):
        self.thisptr = new wrappers.Multiplier()

    def __dealloc__(self):
        del self.thisptr

    def mult(self, n):
        self.thisptr.mult(n)

    property product:
        def __get__(self):
            return self.thisptr.get_product()
~~~

~~~
Makefile:

CPP=g++
CPPFLAGS=-Wall -ansi -pedantic -std=c++11 -fPIC -I./include -I$(ANACONDA_DIR)/include/python2.7
LDFLAGS=-shared -L$(ANACONDA_DIR)/lib -lpython2.7

all: wrappers.so

wrappers.so: Adder.o Multiplier.o wrappers.o
    $(CPP) $(LDFLAGS) -o $@ $^

src/wrappers.cpp: wrappers.pyx
    cython --cplus -o $@ $<

%.o: src/%.cpp include/%.hpp
    $(CPP) $(CPPFLAGS) -c $<

clean:
    rm -f src/wrappers.cpp *.o *.so

~~~