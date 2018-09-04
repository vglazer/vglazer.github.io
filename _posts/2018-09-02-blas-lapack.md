---
layout: post
title:  "BLAS and LAPACK"
date:   2018-09-02 19:41:00
categories: libraries mac osx numerical_analysis linear_algebra scientific_computing
---

## Background
[LAPACK](http://www.netlib.org/lapack/) is a standard library -- written in Fortran -- for 
common linear algebra tasks such as solving systems of linear equations and 
least squares problems, finding eigenvalues and singular values, as well as factorizing 
matrices into decompositions including LU, Cholseky, QR and SVD (see [this page](http://www.netlib.org/lapack/lug/node19.html#chapcontents) 
for a detailed list). LAPACK is leveraged by much scientific computing, engineering, statistics
and financial software, including MATLAB, R, NumPy and GSL.

LAPACK is [open source software](https://en.wikipedia.org/wiki/Open-source_software) and 
can be downloaded from [Netlib](http://www.netlib.org), the numerical software repository, as well as from [GitHub](https://github.com/Reference-LAPACK/lapack-release). It is
[licensed](http://www.netlib.org/lapack/LICENSE.txt) under the [permissive](https://en.wikipedia.org/wiki/Permissive_software_licence) ["3-clause BSD license"](https://opensource.org/licenses/BSD-3-Clause). 
Here is a [plain English explanation](https://tldrlegal.com/license/bsd-3-clause-license-(revised)) of 
this license. A C interface called [LAPACKE](http://www.netlib.org/lapack/lapacke.html) is included with 
LAPACK. To maximize vectorization while abstracting away architectural differences, LAPACK algorithms 
are written in terms of fundamental "building blocks" such as matrix-matrix and matrix-vector 
multiplication. LAPACK specifies a Fortran interface for such operations, called [BLAS](http://www.netlib.org/blas/). 
The equivalent C interface is called [CBLAS](http://www.netlib.org/blas/#_cblas).

LAPACK is designed to efficiently handle dense matrices (mostly non-zero entries) and 
[banded matrices](https://en.wikipedia.org/wiki/Band_matrix) (e.g. [tridiagonal](https://en.wikipedia.org/wiki/Tridiagonal_matrix)). 
General [sparse matrices](https://en.wikipedia.org/wiki/Sparse_matrix) (whose entries are 
mostly zero), which [often come up](https://machinelearningmastery.com/sparse-matrices-for-machine-learning/) 
in Machine Learning applications, require specialized algorithms not included in the standard 
LAPACK distribution.

Note that BLAS is an interface specification, not an implementation. While a 
Fortran reference implementation -- also licensed under the 3-clause BSD license -- is 
available from Netlib and as part of the LAPACK GitHub repo, its use in production is strongly
discouraged. This is because LAPACK's performance is primarily driven 
by how fast the underlying BLAS is, and the reference implementation is not especially fast.
Instead, the idea is for system vendors to provide optimized BLAS implementaions that fully 
exploit whatever support for parallelism their architectures provide (such as [SIMD](https://en.wikipedia.org/wiki/SIMD) instruction sets including 
[MMX](https://en.wikipedia.org/wiki/MMX_(instruction_set)), [SSE](https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions)
and [AVX](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions)). If such a vendor-supplied
BLAS implementation exists for your system, that should be your first choice. 

In practice, vendors often treat LAPACK itself as a reference implementation and re-implement 
portions of it to boost performance, as well as providing various extensions. In particular, 
many vendors supply routines for efficient handling of sparse matrices. See [this page](http://www.netlib.org/utk/people/JackDongarra/la-sw.html) 
for more information.

For the historically minded, the following ["oral history"](http://history.siam.org/oralhistories.htm)
of BLAS may prove of some inteterest.

## Select Vendor Implementations
* A highly-tuned [BLAS](https://software.intel.com/en-us/mkl-developer-reference-c-blas-and-sparse-blas-routines) 
implementation is included with Intel's [MKL](https://software.intel.com/en-us/mkl), 
which also provides optimized implementations of various [LAPACK routines](https://software.intel.com/en-us/mkl-developer-reference-c-lapack-routines). 
MKL is licensed under a permissive license called the [ISSL](https://software.intel.com/en-us/license/intel-simplified-software-license), 
which you can read about [here](https://software.intel.com/en-us/mkl/license-faq). Note, 
however, that Intel does not make the code for the [linear algebra portion](https://software.intel.com/en-us/mkl/features/linear-algebra) of the MKL freely
available. On the other hand, the [Deep Neural Nets-specific portion](https://software.intel.com/en-us/mkl/features/deep-neural-networks) 
is on [GitHub](https://github.com/intel/mkl-dnn), [licensed](https://github.com/intel/mkl-dnn/blob/master/LICENSE) under [version 2.0 of the permissive 
Apache license](https://www.apache.org/licenses/LICENSE-2.0). Additional background on the MKL is available on [this page](https://en.wikipedia.org/wiki/Math_Kernel_Library). 
* AMD has their own [BLAS implementation](https://developer.amd.com/amd-cpu-libraries/blas-library/) called [BLIS](https://github.com/amd/blis),
as well as their own LAPACK-like library called [libflame](https://github.com/amd/libflame). 
Both BLIS and libflame are licensed under the "3-clause BSD license", just like BLAS and LAPACK
(see [here](https://github.com/amd/blis/blob/master/LICENSE) and [here](https://github.com/amd/libflame/blob/master/LICENSE)).
For an in-depth look at BLIS, check out [these](http://www.cs.utexas.edu/users/flame/pubs/blis1_toms_rev3.pdf) [papers](http://www.cs.utexas.edu/users/flame/pubs/blis2_toms_rev3.pdf).
The libflame reference manual can be found [here](http://www.cs.utexas.edu/~flame/web/libflame.pdf).
* Apple provides an optimized [BLAS implementation](https://developer.apple.com/documentation/accelerate/blas) 
as part of its [vecLib](https://developer.apple.com/documentation/accelerate/veclib) library, 
which also includes an optimized LAPACK. vecLib is part of Apple's 
[Accelerate](https://developer.apple.com/documentation/accelerate) framework.
* [Arm Performance Libraries](https://developer.arm.com/products/software-development-tools/hpc/arm-performance-libraries) 
provides optimized BLAS and LAPACK implementations for the ARM architecture. You will need a 
valid Arm Allinea Studio [license](https://developer.arm.com/products/software-development-tools/hpc/arm-compiler-for-hpc/installation/get-license) 
to download it.
* [Cray](https://en.wikipedia.org/wiki/Cray) provides proprietary BLAS and LAPACK implementations 
for their supercomputers as part of [CMSL / LibSci](https://pubs.cray.com/content/S-2529/17.05/xctm-series-programming-environment-user-guide-1705-s-2529/cray-scientific-and-math-libraries-csml). 
You can read more about LibSci [here](http://www.nersc.gov/users/software/programming-libraries/math-libraries/libsci/). 

## Other BLAS Implementations
* [ATLAS](http://math-atlas.sourceforge.net/) is a portable (i.e. architecture-independent) yet 
still quite efficient BLAS implementation. It is generally slower than a BLAS tuned for a 
particular architecture such as MKL, but much faster than Netlib's reference BLAS implementation.
ATLAS uses the same [3-clause BSD license](http://math-atlas.sourceforge.net/faq.html#license) as LAPACK.
It can be obtained from [SourceForge](https://sourceforge.net/projects/math-atlas/files/Stable/).
* [GotoBLAS](https://en.wikipedia.org/wiki/GotoBLAS) was a BLAS optimization hand-optimized for 
the Intel and AMD processors. It is no longer maintained, but can still be downloaded 
[here](https://www.tacc.utexas.edu/research-development/tacc-software/gotoblas2). GotoBLAS is
open source software, licensed under the BSD license.
* [OpenBLAS](https://www.openblas.net/) is a fork of GotoBLAS which is still under active 
development. It is available on [GitHub](https://github.com/xianyi/OpenBLAS), licensed under
the ["3 clause BSD license"](https://github.com/xianyi/OpenBLAS/blob/develop/LICENSE). OpenBLAS
claims to be [about as fast as MKL](https://github.com/xianyi/OpenBLAS/wiki/faq#sandybridge_perf), 
which for many years was only available commercially.
* [uBLAS](https://www.boost.org/doc/libs/1_68_0/libs/numeric/ublas/doc/index.html) is a C++
template class library providing a portable BLAS implementation. It is distributed as part of 
[Boost](https://www.boost.org/). You can read more about the rationale behind uBLAS 
[here](https://www.boost.org/doc/libs/1_68_0/libs/numeric/ublas/doc/overview.html#rationale).
Some tips for getting the most out of uBLAS can be found [here](http://www.crystalclearsoftware.com/cgi-bin/boost_wiki/wiki.pl?Effective_UBLAS).
There is a FAQ [here](https://www.boost.org/doc/libs/1_68_0/libs/numeric/ublas/doc/index.html#6FrequentlyAskedQuestions) 
and another one [here](http://www.crystalclearsoftware.com/cgi-bin/boost_wiki/wiki.pl?Frequently_Asked_Questions_Using_UBLAS).
uBLAS prioritizes portability over performance, like ATLAS, but is slower than ATLAS due to the
"abstraction penalty", and thus even slower than BLAS implementatinos tuned for particular
architectures such as OpenBLAS, MTL or BLIS.

## Which BLAS to Use
Based on the above, I would suggest the following waterfall to maximize performance without
sacrificing redistribution rights:
* If you are on a Mac, use vecLib
* Otherwise, use OpenBLAS if you are on Intel or AMD
* For all other architectures such as ARM, use ATLAS
