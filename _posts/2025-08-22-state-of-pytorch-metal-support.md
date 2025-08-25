---
layout: post
title:  "State of PyTorch Metal support"
date:   2025-08-22 21:36:00
permalink: pytorch-metal-support
categories: metal pytorch llms nanogpt jit osx
---

## Background

- Metal support [blog post](https://pytorch.org/blog/introducing-accelerated-pytorch-training-on-mac/) from 2022
- MPS [backend](https://github.com/pytorch/pytorch/wiki/MPS-Backend)
- If you've used [Ollama](https://ollama.com/), you've taken advantage of this

## Running nanoGPT on Metal

### Setup

- `git clone https://github.com/karpathy/nanoGPT.git`
- `cd nanoGPT`
- `uv init`
- `uv pip install torch numpy transformers datasets tiktoken wandb tqdm`
- `uv run data/shakespeare_char/prepare.py`

### Training

- On the CPU:

     ```
     Overriding: device = cpu
     Overriding: compile = False
     tokens per iteration will be: 16,384
     found vocab_size = 65 (inside data/shakespeare_char/meta.pkl)
     Initializing a new model from scratch
     number of parameters: 10.65M
     num decayed parameter tensors: 26, with 10,740,096 parameters
     num non-decayed parameter tensors: 13, with 4,992 parameters
     using fused AdamW: False
     step 0: train loss 4.2874, val loss 4.2823
     iter 0: loss 4.2655, time 119975.84ms, mfu -100.00%
     iter 10: loss 3.1338, time 1972.52ms, mfu 0.19%
     iter 20: loss 2.7556, time 1962.79ms, mfu 0.19%
     iter 30: loss 2.6094, time 1978.37ms, mfu 0.19%
     iter 40: loss 2.5535, time 2014.34ms, mfu 0.19%
     iter 50: loss 2.5237, time 1985.25ms, mfu 0.19%
     iter 60: loss 2.5046, time 1968.88ms, mfu 0.19%
     iter 70: loss 2.4889, time 1990.69ms, mfu 0.19%
     iter 80: loss 2.4647, time 1960.85ms, mfu 0.19%
     iter 90: loss 2.4586, time 1976.99ms, mfu 0.19%
     iter 100: loss 2.4558, time 1978.22ms, mfu 0.19%
     iter 110: loss 2.4477, time 1970.54ms, mfu 0.19%
     iter 120: loss 2.4365, time 2002.86ms, mfu 0.19%
     iter 130: loss 2.4185, time 1977.23ms, mfu 0.19%
     iter 140: loss 2.4315, time 1990.33ms, mfu 0.19%
     iter 150: loss 2.3838, time 2005.56ms, mfu 0.19%
     iter 160: loss 2.3597, time 1965.39ms, mfu 0.19%
     iter 170: loss 2.3292, time 1970.32ms, mfu 0.19%
     iter 180: loss 2.2825, time 1987.84ms, mfu 0.19%
     iter 190: loss 2.2682, time 1984.00ms, mfu 0.19%
     iter 200: loss 2.2122, time 2012.52ms, mfu 0.19%
     iter 210: loss 2.2021, time 2005.32ms, mfu 0.19%
     ...
     ```

- On mps,  without compiling: `/usr/bin/time uv run train.py config/train_shakespeare_char.py --device=mps --compile=False`

     ```
     Overriding: device = mps
     Overriding: compile = False
     tokens per iteration will be: 16,384
     found vocab_size = 65 (inside data/shakespeare_char/meta.pkl)
     Initializing a new model from scratch
     number of parameters: 10.65M
     num decayed parameter tensors: 26, with 10,740,096 parameters
     num non-decayed parameter tensors: 13, with 4,992 parameters
     using fused AdamW: False
     step 0: train loss 4.2874, val loss 4.2823
     iter 0: loss 4.2639, time 22431.66ms, mfu -100.00%
     iter 10: loss 3.1459, time 226.65ms, mfu 1.64%
     iter 20: loss 2.7319, time 229.56ms, mfu 1.64%

     ...

     iter 4970: loss 0.7901, time 215.81ms, mfu 1.37%
     iter 4980: loss 0.7922, time 216.43ms, mfu 1.40%
     iter 4990: loss 0.8216, time 216.65ms, mfu 1.43%
     step 5000: train loss 0.6186, val loss 1.7028
     iter 5000: loss 0.8151, time 344984.99ms, mfu 1.29%
          4233.60 real        96.24 user        31.10 sys
     ```

- With compiling: `/usr/bin/time uv run train.py config/train_shakespeare_char.py --device=mps --compile=True` or just `/usr/bin/time uv run train.py config/train_shakespeare_char.py --device=mps`
- The good news is that compiling the model works out of the box and does not produce missing kernel errors, but it actually makes the model _slower_ per iteration, in addition to the upfront compilation cost!

     ```
     Overriding: device = mps
     Overriding: compile = True
     tokens per iteration will be: 16,384
     found vocab_size = 65 (inside data/shakespeare_char/meta.pkl)
     Initializing a new model from scratch
     number of parameters: 10.65M
     num decayed parameter tensors: 26, with 10,740,096 parameters
     num non-decayed parameter tensors: 13, with 4,992 parameters
     using fused AdamW: False
     compiling the model... (takes a ~minute)
     step 0: train loss 4.2874, val loss 4.2823
     iter 0: loss 4.2672, time 80274.86ms, mfu -100.00%
     iter 10: loss 3.1471, time 341.82ms, mfu 1.09%
     iter 20: loss 2.7730, time 342.52ms, mfu 1.09%
     iter 30: loss 2.6538, time 336.74ms, mfu 1.09%
     iter 40: loss 2.5960, time 334.44ms, mfu 1.09%
     iter 50: loss 2.5479, time 336.10ms, mfu 1.10%
     iter 60: loss 2.5225, time 333.87ms, mfu 1.10%
     iter 70: loss 2.5075, time 334.01ms, mfu 1.10%
     iter 80: loss 2.5035, time 335.73ms, mfu 1.10%
     iter 90: loss 2.4759, time 334.07ms, mfu 1.10%
     iter 100: loss 2.4703, time 334.01ms, mfu 1.10%
     iter 110: loss 2.4623, time 334.57ms, mfu 1.10%
     iter 120: loss 2.4334, time 335.27ms, mfu 1.10%
     iter 130: loss 2.4249, time 334.62ms, mfu 1.11%
     iter 140: loss 2.4050, time 334.08ms, mfu 1.11%
     iter 150: loss 2.4091, time 334.23ms, mfu 1.11%
     iter 160: loss 2.3787, time 334.89ms, mfu 1.11%
     iter 170: loss 2.3538, time 334.23ms, mfu 1.11%
     iter 180: loss 2.3063, time 334.52ms, mfu 1.11%
     iter 190: loss 2.2496, time 333.95ms, mfu 1.11%
     iter 200: loss 2.2144, time 333.88ms, mfu 1.11%
     iter 210: loss 2.1572, time 334.48ms, mfu 1.11%
     iter 220: loss 2.1547, time 334.71ms, mfu 1.11%
     iter 230: loss 2.0977, time 332.57ms, mfu 1.11%
     iter 240: loss 2.1005, time 309784.28ms, mfu 1.00%
     step 250: train loss 1.9921, val loss 2.0831
     saving checkpoint to out-shakespeare-char
     iter 250: loss 2.0691, time 79104.90ms, mfu 0.90%
     iter 260: loss 1.9987, time 335.19ms, mfu 0.92%
     iter 270: loss 2.0043, time 335.91ms, mfu 0.94%
     iter 280: loss 2.0042, time 336.12ms, mfu 0.96%
     iter 290: loss 1.9525, time 342.01ms, mfu 0.97%
     iter 300: loss 1.9256, time 335.53ms, mfu 0.98%
     iter 310: loss 1.8933, time 335.73ms, mfu 1.00%
     iter 320: loss 1.8855, time 335.90ms, mfu 1.01%
     iter 330: loss 1.8544, time 335.78ms, mfu 1.02%
     iter 340: loss 1.8091, time 335.52ms, mfu 1.03%
     iter 350: loss 1.8614, time 335.77ms, mfu 1.04%

     ...

     ```

### Inference


- `uv run sample.py --out_dir=out-shakespeare-char --device=mps --compile=False`

     ```
     Overriding: out_dir = out-shakespeare-char
     Overriding: device = mps
     Overriding: compile = False
     number of parameters: 10.65M
     Loading meta from data/shakespeare_char/meta.pkl...


     Nurse:
     Madam, what says your friend?

     FRIAR LAURENCE:
     Hang you true, come. Pray, be not, ready.

     DUKE VINCENTIO:
     I am a thousand services shall be done:
     Let's be woeful for your health.

     MARIANA:
     Not of the prince; your honour service
     That proper ensuing you for here, not for the crown;
     For I until you read no loyal person.

     DUKE VINCENTIO:
     Let me speak the matter strange quite him when
     'Tis a ground;' many to do it.' An I will read the sess,
     Which you mean, when you are not this disease, I see
     ---------------


     AUTOLYCUS:
     Which is the day?

     Clown:
     See here, sir, thou hast been honour!
     ```

- [asitop](https://github.com/tlkh/asitop)
- what happens when you try to JIT using Torch.compile?
- General MPS ops coverage [tracking issue](https://github.com/pytorch/pytorch/issues/77764)

## Building Pytorch nightly from source

## More Metal-friendly alternatives to PyTorch

- [tinygrad](https://github.com/tinygrad/tinygrad)
- [mlx](https://github.com/ml-explore/mlx)