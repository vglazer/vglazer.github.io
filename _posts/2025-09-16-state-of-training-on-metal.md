---
layout: post
title:  "State of training on Metal"
date:   2025-09-16 22:00:00
permalink: training-on-metal
categories: metal jax pytorch llms nanogpt tinygrad mlx ane asitop
---

![Training On Metal](/assets/MLTrainingOnMetal.png)

## TL;DR

- Using the nightly pytorch build, I am no longer getting CPU fallback warnings due to missing kernels when training nanoGPT on [Metal](https://developer.apple.com/documentation/MetalPerformanceShaders).
- However &mdash; on my Macbook, at least &mdash; training on Metal is significantly slower with [`torch.compile`](https://docs.pytorch.org/tutorials/intermediate/torch_compile_tutorial.html) than without it (though still faster than training on the CPU).

## Background

- Apple Silicon's unified memory architecture makes the Mac a [potentially attractive platform](https://arxiv.org/pdf/2501.14925) for ML training, particularly when it comes to models too large to fit into memory on a single NVIDIA card.
- To take advantage of this, though, you need an ML framework with good [Metal (MPS)](https://developer.apple.com/documentation/MetalPerformanceShaders) support, ideally one capable of optimizing computational graphs through compilation (JIT or otherwise).
- A Metal backend was [first added to PyTorch](https://pytorch.org/blog/introducing-accelerated-pytorch-training-on-mac/) back in 2022. While much progress has been made since then, MPS Ops coverage is [still incomplete](https://qqaatw.dev/pytorch-mps-ops-coverage/).
  - For example, if you try to run `torch.svd` on MPS, you get this error: "The operator `aten::linalg_svd` is not currently supported on the MPS backend and will fall back to run on the CPU. This may have performance implications.".
  - As of now, [over 70 ops](https://github.com/users/kulinseth/projects/1/views/1) are tagged with either "To triage" or "To be implemented".
  - There is [a Github issue](https://github.com/pytorch/pytorch/issues/77764) you can comment on to drive prioritization.
- Apple has recently released its own ML framework, called [MLX](https://github.com/ml-explore/mlx), which supports not only MPS but also [Neural Engine (ANE)](https://en.wikipedia.org/wiki/Neural_Engine).
  - Unlike [Core ML](https://developer.apple.com/documentation/coreml), which is primary geared towards inference and fine-tuning, MLX is intended for training models from scratch in a flexible way, similar to PyTorch or JAX.
  - There are no tools for automatically converting PyTorch (or JAX) models to MLX the way you can covert models to Core ML with [coremltools](https://github.com/apple/coremltools), though.
  - You can certainly port the training loop manually, but it's [non-trivial](https://github.com/pranavjad/mlx-gpt2). When I tried to get Gemini CLI to port nanoGPT to MLX it kept  getting stuck, even after much prodding.
- [JAX](https://github.com/jax-ml/jax) has historically prioritized TPUs when it comes to non-NVIDIA accelerators, which makes sense given its Google lineage. While support for MPS in JAX is [in the works](https://developer.apple.com/metal/jax/), it's [still experimental](https://github.com/jax-ml/jax?tab=readme-ov-file#supported-platforms) at this point.
- Then there's [tinygrad](https://github.com/tinygrad/tinygrad), which aims to provide first-class support for MPS out of the box. However, as with MLX, there is no way to automatically convert PyTorch models to tinygrad. A number of popular models [have been ported](https://docs.tinygrad.org/showcase/), though. You can see what training looks like [here](https://docs.tinygrad.org/mnist/#training-the-model).

## nanoGPT as a training benchmark

- [nanoGPT](https://github.com/karpathy/nanoGPT) is Adrej Karpathy's from-scratch implementation of GPT-2 in PyTorch.
- There are two main drivers, [`train.py`](https://github.com/karpathy/nanoGPT/blob/master/train.py) for training and [`sample.py`](https://github.com/karpathy/nanoGPT/blob/master/sample.py) for inference.
- The default is to use NVIDIA GPUs and JIT-compile the model into optimized kernels via [`torch.compile`](https://docs.pytorch.org/tutorials/intermediate/torch_compile_tutorial.html), but you can override this using the `--device` and `--compile` command-line switches, respectively.
- When nanoGPT was first released (in 2022), MPS support in PyTorch was still nascent. Training on the Mac was limited to the CPU in practice, so much so that the [quick start section](https://github.com/karpathy/nanoGPT?tab=readme-ov-file#quick-start) of README.md branched on "I have a GPU" vs "I only have a macbook".
- This have steadily improved over time, though, as evidenced by [this github issue](https://github.com/karpathy/nanoGPT/issues/28).

## Training nanoGPT on the Mac

### Setup

This assumes you have [uv](https://docs.astral.sh/uv/) installed:

- `git clone https://github.com/karpathy/nanoGPT.git`
- `cd nanoGPT`
- `uv venv`
- `uv pip install numpy transformers datasets tiktoken wandb tqdm`
- `uv pip install --pre torch --index-url https://download.pytorch.org/whl/nightly/cpu`
- `uv run data/shakespeare_char/prepare.py`

### Metal training without `torch.compile`

```
/usr/bin/time uv run train.py config/train_shakespeare_char.py --device=mps --compile=False

...

step 0: train loss 4.2874, val loss 4.2823
iter 0: loss 4.2639, time 25857.56ms, mfu -100.00%
iter 10: loss 3.1459, time 220.50ms, mfu 1.69%
iter 20: loss 2.7319, time 221.44ms, mfu 1.69%
iter 30: loss 2.6226, time 221.30ms, mfu 1.69%
iter 40: loss 2.5756, time 220.74ms, mfu 1.69%
iter 50: loss 2.5239, time 220.24ms, mfu 1.69%

...

iter 4990: loss 0.8203, time 276.23ms, mfu 1.33%
step 5000: train loss 0.6166, val loss 1.7099
iter 5000: loss 0.8117, time 33010.34ms, mfu 1.19%
     3588.66 real        90.18 user        32.03 sys
```

### Metal training with `torch.compile`

```
/usr/bin/time uv run train.py config/train_shakespeare_char.py --device=mps --compile=True

...

step 0: train loss 4.2874, val loss 4.2823
iter 0: loss 4.2672, time 85292.53ms, mfu -100.00%
iter 10: loss 3.1471, time 304.79ms, mfu 1.22%
iter 20: loss 2.7730, time 305.33ms, mfu 1.22%
iter 30: loss 2.6538, time 306.39ms, mfu 1.22%
iter 40: loss 2.5960, time 306.01ms, mfu 1.22%
iter 50: loss 2.5479, time 305.41ms, mfu 1.22%

...

iter 4990: loss 0.8286, time 329.89ms, mfu 1.11%
step 5000: train loss 0.6221, val loss 1.7106
iter 5000: loss 0.8242, time 1255915.73ms, mfu 1.00%
    39039.74 real        72.21 user        21.28 sys
```

### CPU training

```
uv run train.py config/train_shakespeare_char.py --device=cpu --compile=False

...

step 0: train loss 4.2874, val loss 4.2823
iter 0: loss 4.2655, time 119984.47ms, mfu -100.00%
iter 10: loss 3.1338, time 1973.80ms, mfu 0.19%
iter 20: loss 2.7556, time 1984.34ms, mfu 0.19%
iter 30: loss 2.6094, time 1988.45ms, mfu 0.19%
iter 40: loss 2.5535, time 1983.75ms, mfu 0.19%
iter 50: loss 2.5237, time 2003.05ms, mfu 0.19%
```

### Observations

- I didn't run CPU training to completion, but training on Metal without `torch.compile` is roughly 9 times faster than training on the CPU based on the 1st 50 iterations (221ms vs 1986ms).
- With `torch.compile`, it's only 6.5 times faster (306ms). In order words, turning on `torch.compile` slows training down by a factor of 1.38.
- You can use [asitop](https://github.com/tlkh/asitop) to confirm that GPU usage is at 100% when training on MPS, whether or not `torch.compile` is used.
- CPU+GPU+ANE is stable at 100% without `torch.compile`. With `torch.compile`, it drops slightly below that at times.
