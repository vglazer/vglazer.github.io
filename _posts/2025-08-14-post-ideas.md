---
layout: post
title:  "Post Ideas"
date:   2025-08-14 21:00:00
permalink: post-ideas
categories: linear_algebra linux cryptography random_oracle_model  
---

# The Surprising Fragility of (Classical) Cryptography
- Nowadays you often hear about the security threat posed by quantum computers, at least once we figure out how to build ones large enough to take advantage of Schor's algorithm and the like. There is even an entire research area, "quantum cryptography", where the goal is to design algirhtms that can withstand attack from such computers. But it's not clear how worried one should be about this in practice, though, especially given how far away we are building quantum computers of significant size.
- People sometimes also mention that "classical cryptography" depends P not equalling NP, which haven't been able to prove so far (it's one of the Clay Math Institute's "Millenium Prize" problems, alongside the likes of the Riemann Hypothesis). Still, most people in the theoretical Computer Science community strongly believe that P is in fact not equial to NP. So this isn't necessarily that scary either.
- While it's true that if P = NP then classical proofs of security break down, the situation is in fact much worse than that: 
    - P != NP tells us that there are problems which are hard in the worst case, meaning that there is _at least one_ "hard" instance of the problem of size `n` for every `n`. The canonical NP-Complete problem is SAT, where we know that most instances are easy. In fact, there are SAT solvers which are quite efficient in practice. What's more, many NP-Complete problems can be tractably approximated arbitrarily well, in the sense that we have fully-polynomial time approximation schemes (FPTAS) for them. Examples include KNAPSACK and SUBSET-SUM.
    -  What we really need for classical security to work are problems that are hard _on average_, not _in the worst case_. In other words, we need a "non-neglibible" (in a specific technical sense) number of instance of size `n` to be "hard" for every "n" rather than just one. Unlike NP-Complete problems, there are very few viable candidates for these: historically basicaly just integer factorization (including related problems such as RSA) and "discrete log" (including its elliptic curve variants). More recently, people have also suggested that lattice problems have the required properties. The trouble is that there is no fundamental reason for integer factorization to be hard, even for classical computers (we already know that quantum computers can solve it efficiently). There are plenty of examples where people in the community generally believed a problem was hard but it turned out to be surprisingly easy, PRIMES (i.e. primality testing) being a ... prime ... example (no pun intended).
    - Formally, the assumption we need to be true for classical proofs of security to hold is that one-way functions exist, which is a much stronger assumption then P != NP. [Expand on this point...]
- Maybe also talk about the Random Oracle Model and how some consturctions are only known to be secure in it. Mention the recent, more "practical", uninstantiability result.

# Matrix decompositions refresher
- LU and how it relates to solving systems of linear equations / matrix inversion, Gaussian elimination, determinants, LUP
- Cholesky and how it relates to LU
- QR and how it relates to solving least squares problems and normal equations, pseudoinverse, Gram-Schmidt
- SVD and its many uses, including PCA, singular values VS eigenvalues 
- Why they are important and how to compute them in an efficient (and numerically stable!) way. Also mention the relevance of how easily an algorithm can be parallelized
- Focus on real numbers, ignore complex numbers
- Mention sparse VS dense matrices, though they are out of scope

# Monotonicity-preserving (or "imposing") splines
- Hermite interpolation defined
- Classical methods
- Newer algorithms
- Hyun's "median" approach
- Finance context: Hagan-West

# Different kind of random graphs

# Clever little programs and other useful terminal tricks
- The Classics:
    - `cut -d, -f1-5,10 file.txt`
    - `find . -type f -name '*.py | xargs grep foo`
    - `top`
    - `uptime`
    - `date`
    - `sort`
    - `head`
    - `tail`
    - `file`
    - `wc`
    - `strace`
- Also worth knowing about
    - `sed s/foo/bar/ file.txt`
    - `awk -F, '{print $1 " -> " $NF}'`
    - `tree -d -L 3`
    - `cal -3`
    - `vimdiff`
    - `watch`
    - `strings`
    - `objdump -t`
    - `od`
- The New Kids on the Block:
    - `shellcheck`
    - `rg`
    - `jq`
    - `glow` (with Dracula!)
- Best of the Rest:
    - `htop`
    - `imgcat`
    - `mc` (with Dracula!)
    