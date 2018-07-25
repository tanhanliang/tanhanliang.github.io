---
layout:		post
title: 		Numpy Performance with Intel Math Kernel Library on a Skylake-W Xeon
date: 		2018-07-22
categories:	benchmarks
---

## BLAS, LAPACK, OpenBLAS, ATLAS & Intel MKL
<b><a href="https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms">Basic Linear Algebra Subprograms</a></b> is a specification for low-level matrix operations such as matrix multiplications. The <b><a href="https://en.wikipedia.org/wiki/LAPACK">Linear Algebra Package</a></b> is a collection of higher-level linear algebra operations, for example for finding the eigenvalues of a matrix. 

Various implementations of these collections of operations exist: the standard one that is installed by default with Numpy, the open sourced <b><a href="https://en.wikipedia.org/wiki/OpenBLAS">OpenBLAS</a></b> and <b><a href="https://en.wikipedia.org/wiki/Automatically_Tuned_Linear_Algebra_Software">Automatically Tuned Linear Algebra Software</a></b>, and finally Intel's proprietary <b><a href="https://en.wikipedia.org/wiki/Math_Kernel_Library">Math Kernel Library</a></b>, which is hand optimised for Intel processors. 

The first of these is a generic implementation, meant to run on any hardware platform, and as expected has the worst performance. ATLAS employs computer assisted optimisation by detecting the computation capabilities of the host computer's hardware, such as instruction set extensions (e.g AVX2). Intel's MKL as expected has the <a href="http://markus-beuckelmann.de/blog/boosting-numpy-blas.html">best performance</a>, at least for Intel processors. MKL does not run as well on AMD processors (what a surprise). 

Numpy uses an implementation of the BLAS and LAPACK operations under the hood. Today I'm going to compare the performance of vanilla numpy vs numpy optimised with the Intel MKL.

## Hardware
The processor I'm using is a 8-core/16-thread <a href="https://browser.geekbench.com/v4/cpu/8906116">Xeon W-2140B</a> (base 3.2 GHz, max all-core turbo 3.9 GHz, max single-core turbo 4.2 GHz) that I got super cheap from ebay (in an apparently new condition too). You probably haven't heard about this processor lineup because of Intel's confusing product names. The Skylake-W Xeons are relatively new and were released in 3Q 2017. This one was built specially for the iMac Pro and you won't find it on Ark Intel.

This processor has the AVX-512 instruction set extensions, with two AVX-512 units per physical core. The naming comes from the 512-bit wide registers in each CPU core, which can operate on 16 single-precision (16x32=512) floating point numbers at a time. Most Intel and AMD processors come with AVX2 instruction sets, which contrary to the name uses 256-bit wide registers and typically come with one 256-bit wide register per core.

Given that I briefly discuss thermal performance of the CPU, I should also mention that I am using the <a href="https://noctua.at/en/tdp-guide">Noctua NH-C14s</a> CPU cooler (fan at 1100-1200rpm), rated for 'low overclocking potential' on a 140 watt CPU. I am using a 120 watt CPU without overclocking, so this cooler is overkill. 

## View numpy configuration
To see which libraries `numpy` is linked against:
~~~ shell
python3 -c 'import numpy as np; np.show_config()'
~~~

I used `virtualenv` to isolate the installations.

## Installing numpy with MKL
Not so long ago, one had to do this manually. Fortunately, `numpy` with MKL is available on `pip` and `pip3`. 
~~~ shell
pip3 install intel-numpy
~~~

## Installing numpy with default BLAS and LAPACK implementations
The version of `numpy` installed through `apt-get` appears to come with this. However, this will affect your global `python` installation. I do not know how to install this version in a `virtualenv`. I doubt that you would want `numpy` to use this anyway.
~~~ shell
sudo apt-get install numpy
~~~ 

## Installing numpy with OpenBLAS
I did not plan to compare OpenBLAS with MKL in this post, but installing `numpy` through `pip3` in a `virtualenv` links `numpy` with OpenBLAS for me.
~~~ shell
pip3 install numpy
~~~

## Benchmark and results
The <a href="https://gist.github.com/markus-beuckelmann/8bc25531b11158431a5b09a45abd6276">benchmark script</a> I'll be using is one Markus Beuckelmann wrote. It has several benchmarks:
1. Matrix multiplication
2. Vector multiplication
3. <a href="https://en.wikipedia.org/wiki/Singular-value_decomposition">Singular value decomposition (factorisation of a matrix)</a>
4. <a href="https://en.wikipedia.org/wiki/Cholesky_decomposition">Cholesky decomposition</a>
5. Computation of eigenvalues of a matrix

### Default implementations of BLAS and LAPACK. 
<figure>
  <img src="/assets/2018-07-22-numpy-vs-intel-numpy/default-blas.png"/>
</figure>
A quick glance at `top` shows me that only 1 thread is utilised fully (100% load). 

### OpenBLAS:
<figure>
  <img src="/assets/2018-07-22-numpy-vs-intel-numpy/openblas.png"/>
</figure>
`top` shows that all 16 threads are utilised fully.

### MKL:
<figure>
  <img src="/assets/2018-07-22-numpy-vs-intel-numpy/mkl.png"/>
</figure>
Perhaps surprisingly, only 8 threads are used fully. 

The results show a huge difference. MKL and OpenBLAS are up to 100-200 times faster than generic BLAS and LAPACK implementations (for matrix multiplications). MKL is up to 1.8 times faster than OpenBLAS, though on some workloads their performance is similar. 

## Thermal behaviour of CPU
To try and detect any performance degradation under sustained load due to heat buildup, I ran the first test, matrix multiplication, for 2000 cycles instead of 20, using MKL and OpenBLAS. This takes about 10 minutes using MKL, and 18 using OpenBLAS. 

Using MKL, CPU core temperature plateaued at an average of 65<sup>o</sup>C, with a maximum of 73<sup>o</sup>C. No performance degradation. 

Using OpenBLAS, CPU core temperature plateaued at an average of 68<sup>o</sup>C, with a maximum of 71<sup>o</sup>C. No performance degradation observed as well. 

These temperatures are a little on the hot side even though my CPU cooler is overkill for the load. The stock Intel cooler would probably perform terribly.  

## Reality
These benchmarks were run under optimal conditions, with no other resource intensive programs running. The only other programs running at the same time were `top`, `psensor` and Sublime text editor. Under a real workload, Python will probably not be able to use all cores on the CPU, and the real speedup over generic BLAS and LAPACK will be less. Also, the real speedup of a particular application will depend on the portion spent running `numpy` calculations, as dictated by <a href="https://en.wikipedia.org/wiki/Amdahl%27s_law">Amdahl's law</a>.

## References
1. <a href="http://markus-beuckelmann.de/blog/boosting-numpy-blas.html">Boosting numpy: Why BLAS matters (Markus Beuckelmann)</a>
2. <a href="https://stackoverflow.com/questions/17858104/what-is-the-relation-between-blas-lapack-and-atlas">What is the relation between BLAS, LAPACK and ATLAS? (Stackoverflow)</a>
3. <a href="https://discourse.julialang.org/t/openblas-is-faster-than-intel-mkl-on-amd-hardware-ryzen/8033">OpenBLAS is faster than Intel MKL on AMD Hardware (Julia)</a>
4. Wikipedia
