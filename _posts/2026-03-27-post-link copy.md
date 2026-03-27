---
title: "Project Euler #001: Multiples of 3 or 5"
categories:
  - Blog
  - Project Euler
tags:
  - python
  - arithmetic
  - divisibility
  - generators
link: https://projecteuler.net/problem=1
---

> If we list all the natural numbers below 10 that are multiples of 3 or 5, we get 3, 5, 6 and 9. The sum of these multiples is 23. Find the sum of all the multiples of 3 or 5 below 1000.

## The Mathematics

We want every integer `x` in the range `[1, 999]` such that `3 | x` or `5 | x` (read: "3 divides x" or "5 divides x").

The elegant closed-form approach uses the **inclusion–exclusion principle**. Define `S(k, n)` as the sum of all multiples of `k` strictly below `n`. Since these form an arithmetic series `k, 2k, 3k, …` with `p = ⌊(n−1)/k⌋` terms:

## The Sage Solution

```python

# S(k, n) = k · p · (p + 1) / 2

def S(k, n):
    p = (n-1) // k
    return k * p * (p + 1) / 2

S(3, 1_000) + S(5, 1_000) - S(15, 1_000)

> 233168
```
The answer is then `S(3, 1,000) + S(5, 1,000) − S(15, 1,000)` — we subtract multiples of 15 because they were counted twice (once as multiples of 3, once as multiples of 5). This runs in **O(1)** time regardless of `n`.

## The Python Solution

The one-liner below takes a more direct, Pythonic approach using a generator expression inside `sum()`:

```python
# "below 1_000" means range(1_000)
def answer(n: int) -> int:
  return sum(x for x in range(1_000) if (x % 3) * (x % 5) == 0)
print(answer(1_000))  # 233168
```

## Why the Trick Works

The condition `(x % 3) * (x % 5) == 0` exploits the **zero-product property**: a product of integers is zero if and only if at least one factor is zero. So the expression is `True` whenever `x % 3 == 0` *or* `x % 5 == 0` — exactly our condition — without needing the `or` keyword.

Compared to the O(1) formula this is **O(n)**, but for `n = 1,000` the difference is imperceptible. Python's generator expression also avoids materialising a list in memory; it streams values one by one into `sum()`.

**Answer (n = 1000): 233168**
