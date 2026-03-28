---
title: "Project Euler #003: Largest Prime Factor"
categories:
  - Blog
  - Project Euler
tags:
  - python
  - primes
  - factorisation
  - recursion
  - generators
  - type hints
link: https://projecteuler.net/problem=3
---

> The prime factors of 13195 are 5, 7, 13 and 29. What is the largest prime factor of the number 600851475143?

## The Mathematics

The **Fundamental Theorem of Arithmetic** guarantees that every integer greater than 1 has a unique prime factorisation. Our strategy is to repeatedly extract the *smallest* prime factor of `n`, yielding each one, until nothing remains. The last factor yielded will be the largest.

The key efficiency insight: if `n` has no divisor in the range `[2, √n]`, then `n` is prime. This bounds our search at `ceil(sqrt(n))`, giving **O(√n)** trial divisions per call — fast even for numbers in the hundreds of billions.

## The Python Solution

The script decomposes the problem into three clean, single-responsibility functions:

```python
"""
Project Euler Problem 3 - Largest Prime Factor.

This module contains functions to find the largest prime factor of a number.
It includes utilities for finding the smallest divisor and generating prime factors.
"""

from typing import Iterator
from math import (ceil, sqrt)


def smallest_divisor(number: int) -> int:
    """
    Returns the smallest divisor of number bigger than 1.
    For example 
    smallest_divisor(100) = 2, 
    smallest_divisor(15) = 3, 
    smallest_divisor(125) = 5.
    """
    assert (number > 1), f"Not a valid input number = {number} must be > 1"
    upper_bound = ceil(sqrt(number))
    if number % 2 == 0:
        return 2
    if number % 3 == 0:
        return 3
    divisors = (d for d in range(5, upper_bound + 1, 2) if number % d == 0)
    try:
        return next(divisors)
    except StopIteration:
        return number


def prime_factors(number: int) -> Iterator[int]:
    """

    :param number: Integer > 1.
    :return: Returns a generator of the prime factors of number.
    """
    if number > 1:
        prime = smallest_divisor(number)
        yield prime
        yield from prime_factors(number // prime)


def answer(number: int) -> int:
    """
    Return the largest prime factor of the given number.

    This function solves Project Euler problem 3 by finding all prime factors
    of the number and returning the largest one.

    Parameters
    ----------
    number : int
        The number to find the largest prime factor of.

    Returns
    -------
    int
        The largest prime factor of the input number.

    Examples
    --------
    >>> answer(13195)
    29
    """
    return list(prime_factors(number))[-1]


print(answer(600_851_475_143))
```

## How `smallest_divisor` Works

The function finds the smallest prime factor of `number` in three steps:

1. **Quick checks for 2 and 3.** These cover all even numbers and multiples of 3 up front, before the main loop.
2. **Odd candidates only.** The generator `range(5, ub + 1, 2)` steps by 2, skipping all even numbers. Since 2 has already been ruled out as a factor, no even number can divide `number`, so this halves the remaining search space.
3. **Early exit via `next()`.** The generator `g` is lazy — it produces candidates on demand. Calling `next(g)` retrieves only the *first* divisor found and stops immediately. If no divisor exists, the `StopIteration` exception is caught and `number` itself is returned, indicating `number` is prime.

## How `prime_factors` Works — Recursion and `yield from`

`prime_factors` is a **recursive generator**. At each call it:

1. Finds the smallest divisor `p` of `number` and `yield`s it.
2. Uses `yield from prime_factors(number // p)` to recursively yield all prime factors of the reduced number.

`yield from` is Python 3's way of delegating to a sub-generator — it forwards every value the recursive call produces directly to the caller, without needing an explicit loop. The recursion bottoms out when `number == 1`, at which point the `if number > 1` guard is `False` and the generator returns.

For example, factorising 12:

```
prime_factors(12)  →  yields 2, then delegates to
  prime_factors(6) →  yields 2, then delegates to
    prime_factors(3) →  yields 3, then delegates to
      prime_factors(1) →  base case, stops
```

Result: `[2, 2, 3]`

## Type Hints and `Iterator`

The signature `def prime_factors(n: int) -> Iterator[int]` uses Python's `typing` module to declare that this function returns an iterator of integers. This is the correct annotation for a generator function — it signals to type checkers (and human readers) that the return value is not a list but something to be consumed lazily. In Python 3.9+, `Iterator` can be imported directly from `collections.abc` instead.

## Putting It Together

```python
def answer(number: int) -> int:
    return list(prime_factors(number))[-1]
```

`prime_factors` yields factors in ascending order because `smallest_divisor` always returns the *smallest* factor first. Collecting them into a list and taking the last element `[-1]` gives the largest prime factor directly.

For `600_851_475_143`, the full factorisation is `71 × 839 × 1471 × 6857`, so the answer is the final element: **6857**.

**Answer: 6857**
