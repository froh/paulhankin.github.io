---
layout: post
title: Everything you know about complexity is wrong
---

Which of these is false? The run-time of merge-sort is $$O(n\mathrm{log}\,n)$$.
It's asymptotically optimal. There's
no known algorithm for solving NP problems in polynomial time.

I expect your answer, like nearly any educated programmer,
is that they're all true as long as the model of computation
is sufficiently close to a real computer, and not a quantum
computer or able to perform arbitrary operations in parallel (possibly
involving sticks of spaghetti).

This post will explore some common models of computing, but discover
that one or more of the statements are false in all of them.

<!--more-->

Flawed model 1: RAM is fixed-sized words
--------------

The model we'll look at is a pragmatic one. Here, every object is assumed to fit into a 64-bit
word of RAM.

This is immediately flawed: an array of $$n$$ items can only contain $$2^{64}$$ different
values, which makes sorting possible in $$O(n)$$ time using a bucket sort with $$2^{64}$$ buckets.

A more subtle problem here is that the registers of our machine must also be 64-bit. That means
that there's a finite (albeit huge) range of memory available to programs, which means that
we can't even represent large arrays.

An obvious generalisation of this model is the "real computer" abstraction, which stores
data across multiple words. That's the next model.

Flawed model 2: RAM is bytes
--------------

Here, we allow data to be stored across multiple fixed-sized locations in memory, just like a real computer.
The maximum value stored in a fixed-size location is irrelevant to the theory, so let's assume 8-bit bytes
so our model as much like a real computer as possible.

Here, assuming we have a array of $$n$$ items each with $$O(\mathrm{log}\, n)$$ bits (which is necessary, otherwise too
many of them will be the same), we can sort in $$O(n\mathrm{log}\,n)$$ time using radix sort.

If, however, we try mergesort, we get that each comparison takes $$O(\mathrm{log}\,n)$$ time,
yielding an $$O(n(\mathrm{log}\,n)^2)$$ run-time.

Again like the fixed-sized word model, we have the additional subtelty about how registers work in this model.
They need to be able to store arbitrarily large integers, and it's not obvious how to design the machine
so that adding $$n$$ to a register is $$O(1)$$ (which is necessary for random access into an array), whilst
avoiding adding a computational backdoor that allows programs to use registers instead of RAM, giving our
machine the power of the next model.

Flawed model 3: RAM is arbitrary integers.
---------------

This is probably the most obvious abstraction that overcomes the problems in the first two models. Here,
every cell can contain an arbitrary integer.

Surprisingly, this model can solve any PSPACE problem in polynomial time [[1]][ref-pspace]. This uses the trick
that we can pack lots of data into a single register and then, much like a SIMD instruction on a real
processor, perform multiple operations in parallel. Unlike a SIMD instruction though, this machine can
perform arbitrarily many such operations.

[ref-pspace]: http://link.springer.com/chapter/10.1007%2F3-540-09510-1_42

That we can solve any PSPACE problem in polynomial time invalidates one of our assumptions, because every
NP problem is in PSPACE.

We could try to adjust the costs of arithmetic to be proportional to the number of bits used (called
the "bit cost" model). That fixes the PSPACE $$=$$ P problem, but like the
"RAM is bytes" model, we can't use merge-sort to sort $$n$$ things in $$O(n\mathrm{log}\,n)$$ time.

Flawed model 4: RAM is variable-sized words.
-------

This model (which is sometimes used by theoreticians) models memory as words each of $$w$$ bits. However,
unlike the "RAM is fixed sized words" model, this $$w$$ depends on the size of the input to a problem, and
in fact for a problem of size $$n$$, $$w = O(\mathrm{log}\, n)$$.

This model is pretty close to making our three facts true: merge sort runs in $$O(n\mathrm{log}\,n)$$ time,
and there's no known algorithms for solving NP problems. Unfortunately though, like the
arbitrary integer model, it can be exploited to do an unreasonable amount of computation with the
wide memory locations, yielding sorting algorithms that are faster that $$O(n\mathrm{log}\,n)$$.

To do that, we can build a fusion tree [[2]][ref-fusion] which is a search
tree that can perform queries in $$O(\mathrm{log\,log}\,n)$$ time, and that can be used to sort
an array of $$n$$ items in $$O(n\mathrm{log}\,n/\mathrm{log\,log}\,n)$$ time.

[ref-fusion]: http://en.wikipedia.org/wiki/Fusion_tree

Conclusion
--------

The gap between computer science and programming theory is wider than it looks.

As practicing programmers, we don't really care about asymptotic performance of algorithms.
When we say "this algorithm is $$O(n^2)$$" we mean something more like "until $$n$$ gets unreasonably
large, and maybe excluding a handful of small edge-cases, the run-time is approximately some smallish
constant times $$n^2$$." This is quite like the definition of big-$$O$$, but differs as it both imposes an
upper limit to $$n$$ and requires the lower limit where edge-cases are ignored to be small.

Confusing this informal notion with big-$$O$$ isn't bad, as in practice it gives us a predictive
model for how algorithms will perform. But let's not assume that this is scientific.
