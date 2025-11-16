---
layout: post
title:  "Mojo First Steps"
date:   2025-11-15 17:00:00
permalink: mojo-first-steps
categories: Metal Mojo MPS ROCm Triton Python GPUs
---

![Mojo](/assets/mojo.png)

## TL;DR

* Mojo is a new, Python-like JIT-complied programming language from [Chris Lattner](https://en.wikipedia.org/wiki/Chris_Lattner) of LLVM (and Swift) fame.
* Read this Chris's excellent [series of blog posts](https://www.modular.com/democratizing-ai-compute) for some background.
* The basic idea is to have the convenience as ease of use of [Triton](https://openai.com/index/triton/), but supporting non-CUDA backends such as AMD's [ROCm](https://en.wikipedia.org/wiki/ROCm) and Apple's [Metal](https://en.wikipedia.org/wiki/Metal_(API)).
* Moreover, there should be no performance penalty over hand-optimized kernels.
* Have a look at [Get started with Mojo](https://docs.modular.com/mojo/manual/get-started) as well as [Get started with GPU programming](https://docs.modular.com/mojo/manual/gpu/intro-tutorial).

## State Of Metal Support

* Metal (MPS) support is fairly new to Mojo. It was announced in [this post](https://forum.modular.com/t/apple-silicon-gpu-support-in-mojo/2295) from Brad Larson back in September.
* Under the covers Mojo makes use of [metal-cpp](https://github.com/bkaradzic/metal-cpp), the C++ drop-in alternative to Metal's Objective C headers.
* Run `xcrun -find metallib`. It should return something like `/var/run/com.apple.security.cryptexd/mnt/com.apple.MobileAsset.MetalToolchain-v17.2.54.0.Nf6rON/Metal.xctoolchain/usr/bin/metallib`.
* If you get an error, launch XCode, navigate to XCode->Settings->Components and click the Get button next to Metal Toolchain under Other Components. After the Metal Toolchain finishes downloading, run `xcrun -find metallib` again to confirm everything works.

## First Steps

* Install [Pixi](https://github.com/prefix-dev/pixi): `brew install pixi` or `curl -fsSL https://pixi.sh/install.sh | sh`. Pixi is like [uv](https://docs.astral.sh/uv/), but it lets you use Conda packages.
* Clone the [Mojo repo](https://github.com/modular/modular/).
* cd [`examples/mojo/gpu_functions`](https://github.com/modular/modular/tree/main/examples/mojo/gpu-functions) (there is a `pixi.toml` there that specifies the necessary deps).
* Use your favorite editor to create a file called `accelerator.mojo` with the following contents:

```python
from sys import has_accelerator
from gpu.host import DeviceContext

def main():
    @parameter
    if not has_accelerator():
        print("No compatible GPU found")
    else:
        # Get the context for the attached GPU
        ctx = DeviceContext()
        print("Found GPU:", ctx.name())
```

* Run `pixi run mojo accelerator.mojo`. This will JIT-compile and run `accelerator.mojo`. On a Mac, the output should be something like "Found GPU: Apple M3 Max".
* You can also use `mojo` to compile standalone executables, like so: `pixi run mojo build mandelbrot.mojo`. They can then be run directly: `./mandelbrot`.
* The rest of the examples in `gpu-functions` should also work, with the exception of [reduction.mojo](https://github.com/modular/modular/blob/main/examples/mojo/gpu-functions/reduction.mojo). That one is too NVIDIA-specifc.
