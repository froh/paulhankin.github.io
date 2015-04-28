---
layout: post
title: Everything you know about complexity is wrong
---

A comparison-based sort of $$n$$ objects can run in at best $$O(n\,\mathrm{log}\,n)$$ time. A hash
table stores $$O(n)$$ objects in $$O(n)$$ space and lookups
take amortized $$O(1)$$ time. There's almost nothing that programmers universally agree on, but
these basic facts of complexity are considered basic truths.
Unfortunately they're wrong.

Complexity theory, as used by practicing programmers, is built on an abstraction
of an idealized, but unspecified, model of computation. We assume that our unspecified
model of computation behaves more-or-less like our actual computers, and that theoretical
results can be translated from theory to practice. In this
we'll take a closer look at what the model of computation is that bridges the theory-practice
gap. What we'll find is that if the model of computation resembles a real computer the complexity
bounds we know aren't achievable, and if the model is made more powerful then
our optimal algorithms are no longer optimal, and in fact quite reasonable-looking
models are very unreasonably powerful. Thus, while theoretical results about complexity do,
in practice, translate XXXXXXXXXXXXXXXXXxxx

Flawed model 1: RAM is fixed-sized words
--------------

The first flawed model is a pragmatic one. Here, every object is assumed to fit into a 64-bit
word of RAM.

This is immediately flawed: an array of $$n$$ items can only contain $$2^{64}$$ different
values, which makes sorting possible in $$O(n)$$ time using a bucket sort with $$2^{64}$$ buckets.

Similarly, a hash table's hashes will, in this model, be 64-bit values. So there'll be
only $$2^{64}$$ different buckets in the hash-table, and the performance in the limit
will tend to the performance of the code that deals with keys that collide, which will
be $$O(n)$$ if linear chaining is used, or $$O(\mathrm{log}\,n)$$ if a balanced tree is used.

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

Here, assuming we have a list of $$n$$ items each with $$O(\mathrm{log}\, n)$$ bits (which is necessary, otherwise too
many of them will be the same),  we can sort in $$O(n\mathrm{log}\,n)$$ time using radix sort. If we try to use
an "optimal" algorithm like mergesort or heapsort, we'll find that each comparison takes $$O(\mathrm{log}\,n)$$ time,
yielding an $$O(n(\mathrm{log}\,n)^2)$$ run-time.

Similarly, the hashes used in our hash table must be $$O(\mathrm{log}\, n)$$ bits, which necessarily will take $$O(\mathrm{log}\, n)$$ time to create, so lookups will be at best $$O(\mathrm{log}\, n)$$ time.

Again like the fixed-sized word model, we have the additional subtelty about how registers work in this model.
They need to be able to store arbitrarily large integers, and it's not obvious how to design the machine
so that adding $$n$$ to a register is $$O(1)$$ (which is necessary for random access into an array), whilst
avoiding adding a computational backdoor that allows programs to use registers instead of RAM, giving our
machine the power of the next model.

Flawed model 3: RAM is arbitrary integers.
---------------

The most obvious flaw of this model is that any amount of data can be stored in a single memory location.

A worse flaw is that such a model can solve any PSPACE problem in polynomial time [[1]][ref-pspace]. This uses the trick
that we can pack lots of data into a single register and then, much like a SIMD instruction on a real
processor, perform multiple operations in parallel. Unlike a SIMD instruction though, this machine can
perform arbitrarily many such operations.

[ref-pspace]: http://link.springer.com/chapter/10.1007%2F3-540-09510-1_42

That we can solve any PSPACE problem in polynomial time is unreasonable: PSPACE contains all NP problems.

One can fix these problems by adjusting the costs to be proportional to the number of bits used. That
reduces the power of the model back to reasonable levels, but we've now got costs effectively the same as the
"RAM is bytes" model, which we've already showed is flawed.

Flawed model 4: RAM is variable-sized words.
-------

This model (which is sometimes used by theoreticians) models memory as words each of $$w$$ bits. However,
unlike the "RAM is fixed sized words" model, this $$w$$ depends on the size of the input to a problem, and
in fact for a problem of size $$n$$, $$w = O(\mathrm{log}\, n)$$.

This model is pretty close to making our basic complexity facts true. Unfortunately though, like the
arbitrary integer model, it can be exploited to do an unreasonable amount of computation with the
wide memory locations, yielding algorithms that are more efficient than what we believe is optimal.

For example, in this model a fusion tree [[2]][ref-fusion] is a search
tree that can perform queries in $$O(\mathrm{log\,log}\,n)$$ time, and they can be used to sort
an array of $$n$$ items in $$O(n\mathrm{log}\,n/\mathrm{log\,log}\,n)$$ time.

[ref-fusion]: http://en.wikipedia.org/wiki/Fusion_tree

Solution
--------

As practicing programmers, we don't actually care about asymptotic performance of algorithms.
When we say "this algorithm is $$O(n^2)$$" we mean something more like "until $$n$$ gets unreasonably
large, and maybe excluding a handful of small edge-cases, the run-time is less than some constant times $$n^2$$."
This is quite like the definition of big-$$O$$, but differs as it both imposes an upper limit to $$n$$
and requires the lower limit where edge-cases are ignored to be small.

With this informal notion, we can happily use the pragmatic model 1 where everything fits into a word, or
the byte-sized RAM model 2 where we can treat the amount of RAM as bounded.

I do wish we could avoid using big-$$O$$ for this informal notion, since we've seen that there's an unbridgeable
gap between the informal notion we're using and the theoretical definition we'd like to assume is equivalent.
