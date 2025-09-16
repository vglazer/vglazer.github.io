---
layout: post
title:  "State of PyTorch Metal support"
date:   2025-09-02 15:00:00
permalink: pytorch-metal-support
categories: metal pytorch llms nanogpt jit osx mlx
---

## TL;DR

- PyTorch supports Metal pretty well these days.
- Previously, I've experienced issues with [TorchInductor](https://dev-discuss.pytorch.org/t/torchinductor-a-pytorch-native-compiler-with-define-by-run-ir-and-symbolic-shapes/747)'s Metal backend when using `torch.compile` (as have [others](https://github.com/pytorch/pytorch/issues/152155)).
- Using the nightly pytorch build, I am no longer getting errors when training e.g. nanoGPT on Metal with compilation enabled.
- However, on my machine (a MacBook Pro M3 Max with 64 GB RAM, running Sequoia 15.6.1) training on Metal is significantly slower with compilation compared to without, which is surprising.
- [asitop](https://github.com/tlkh/asitop) is an [htop](https://htop.dev/)-like utility for the Mac which lets you monitor GPU and ANE usage.

## Background

- Apple Silicon's "unified memory" architecture makes the Mac a [potentially attractive platform](https://arxiv.org/pdf/2501.14925) for independent ML researchers, particularly when it comes to models too large to fit into VRAM for consumer NVIDIA cards.
- To take advantage of this you need an ML framework with good Metal (MPS) support, which was [first added to PyTorch](https://pytorch.org/blog/introducing-accelerated-pytorch-training-on-mac/) back in 2022. While good progress has been made since then, MPS Ops coverage is [still incomplete](https://qqaatw.dev/pytorch-mps-ops-coverage/). As of now, [over 70 ops](https://github.com/users/kulinseth/projects/1/views/1) are tagged with either "To triage" or "To be implemented". There is [a Github issue](https://github.com/pytorch/pytorch/issues/77764) you can comment on to help with prioritization.
- Apple has since released their own ML framework, called [MLX](https://github.com/ml-explore/mlx), which supports not only MPS but also Apple's Neural Engine (ANE). However, there are no tools for automatically converting PyTorch models to MLX and doing so manually is [non-trivial](https://github.com/pranavjad/mlx-gpt2). Moreover, when I tried to get [Gemini CLI](https://github.com/google-gemini/gemini-cli) to help, it kept getting stuck even after reading the relevant documentation. This might have been due to how I was prompting it, though.
- [JAX](https://github.com/jax-ml/jax) has historically prioritized TPUs when it comes to non-NVIDIA accelerators, which makes sense given its lineage. Support for MPS in JAX is [still experimental](https://github.com/jax-ml/jax?tab=readme-ov-file#supported-platforms) at this point, but [improving](https://developer.apple.com/metal/jax/).
- There is also [tinygrad](https://github.com/tinygrad/tinygrad), a cross-platform ML framework which aims to provide first-class support for MPS out of the box. Tinygrad is pretty niche, however, and its main benefit is simplicity rather than performance.
- Given its relative dominance, it would be good to get a sense of how well PyTorch supports training on Metal these days.

## nanoGPT as a yardstick

- [nanoGPT](https://github.com/karpathy/nanoGPT) is Adrej Karpathy's from-scratch implementation of GPT-2 in PyTorch.
- There are two main drivers, [`train.py`](https://github.com/karpathy/nanoGPT/blob/master/train.py) for training and [`sample.py`](https://github.com/karpathy/nanoGPT/blob/master/sample.py) for inference. 
- The default is to use NVIDIA GPUs and JIT-compile the model into optimized kernels via [`torch.compile`](https://docs.pytorch.org/tutorials/intermediate/torch_compile_tutorial.html), but you can override this using the `--device` and `--compile` command-line switches, respectively.
- When nanoGPT was first released (in 2022), MPS support in PyTorch was still nascent. Training on the Mac was limited to the CPU in practice, so much so that the [quick start section](https://github.com/karpathy/nanoGPT?tab=readme-ov-file#quick-start) of README.md branched on "I have a GPU" vs "I only have a macbook".
- This have steadily improved over time, though, as evidenced by [this github issue](https://github.com/karpathy/nanoGPT/issues/28).

## Running nanoGPT on Metal

### Setup

This assumes you have [uv](https://docs.astral.sh/uv/) installed:

- `git clone https://github.com/karpathy/nanoGPT.git`
- `cd nanoGPT`
- `uv venv`
- `uv pip install numpy transformers datasets tiktoken wandb tqdm`
- `uv pip install --pre torch --index-url https://download.pytorch.org/whl/nightly/cpu`
- `uv run data/shakespeare_char/prepare.py`

### Training (MPS, without compilation)

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
iter 60: loss 2.5127, time 220.67ms, mfu 1.69%
iter 70: loss 2.4897, time 222.41ms, mfu 1.69%
iter 80: loss 2.4936, time 220.38ms, mfu 1.69%
iter 90: loss 2.4706, time 220.58ms, mfu 1.69%
iter 100: loss 2.4636, time 221.86ms, mfu 1.69%
<snip>
iter 4900: loss 0.8038, time 275.84ms, mfu 1.30%
iter 4910: loss 0.8293, time 281.13ms, mfu 1.30%
iter 4920: loss 0.8117, time 274.68ms, mfu 1.30%
iter 4930: loss 0.8076, time 276.59ms, mfu 1.31%
iter 4940: loss 0.7992, time 280.57ms, mfu 1.31%
iter 4950: loss 0.8223, time 276.39ms, mfu 1.31%
iter 4960: loss 0.8129, time 275.30ms, mfu 1.32%
iter 4970: loss 0.7781, time 274.56ms, mfu 1.32%
iter 4980: loss 0.7938, time 279.08ms, mfu 1.32%
iter 4990: loss 0.8203, time 276.23ms, mfu 1.33%
step 5000: train loss 0.6166, val loss 1.7099
iter 5000: loss 0.8117, time 33010.34ms, mfu 1.19%
     3588.66 real        90.18 user        32.03 sys
```

### Training (MPS, with compilation)

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
iter 60: loss 2.5225, time 306.27ms, mfu 1.22%
iter 70: loss 2.5075, time 305.45ms, mfu 1.22%
iter 80: loss 2.5035, time 305.19ms, mfu 1.22%
iter 90: loss 2.4759, time 306.41ms, mfu 1.22%
iter 100: loss 2.4703, time 306.06ms, mfu 1.22%
<snip>
iter 4900: loss 0.8092, time 309.37ms, mfu 1.13%
iter 4910: loss 0.8279, time 309.40ms, mfu 1.14%
iter 4920: loss 0.8184, time 307.53ms, mfu 1.14%
iter 4930: loss 0.8153, time 318.51ms, mfu 1.15%
iter 4940: loss 0.8012, time 307.91ms, mfu 1.15%
iter 4950: loss 0.8239, time 305.14ms, mfu 1.16%
iter 4960: loss 0.8355, time 913.04ms, mfu 1.08%
iter 4970: loss 0.7860, time 306.21ms, mfu 1.10%
iter 4980: loss 0.7914, time 314.64ms, mfu 1.11%
iter 4990: loss 0.8286, time 329.89ms, mfu 1.11%
step 5000: train loss 0.6221, val loss 1.7106
iter 5000: loss 0.8242, time 1255915.73ms, mfu 1.00%
    39039.74 real        72.21 user        21.28 sys
```

### Training (CPU, no compilation)

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

- You can use [asitop](https://github.com/tlkh/asitop) to confirm that GPU usage is at 100% when training on MPS, whether or not compilation is enabled.
- CPU+GPU+ANE is stable at 100% without compilation. With compilation, it generally hovers around 100% though drops slightly below that at times.
- I didn't run CPU training to completion, since it would take too long for a model of this size. It's only there for comparison purposes.
- If you look at iterations 10-50, the average time is about 221ms without compilation, 306ms with compilation and 1986ms on the CPU 
