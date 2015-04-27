---
layout: post
title: An integer formula for Fibonacci numbers
---

This code, somewhat surprisingly, generates Fibonacci numbers.
{% highlight python %}
    def fib(n):
        return (4 << n*(3+n)) // ((4 << 2*n) - (2 << n) - 1) & ((2 << n) - 1)
{% endhighlight %}
In this blog post, I'll explain where it comes from and how it works.
<!--more-->

Before getting to explaining, I'll give a whirlwind background overview of
Fibonacci numbers and how to compute them. If you're already a maths whiz, you can skip
most of the introduction, quickly
skim the section "Generating functions" and then read "An integer formula".

Overview
--------

The Fibonacci numbers are a well-known sequence of numbers:

$$1, 1, 2, 3, 5, 8, 13, 21, 34, 55, ...$$

The $$n$$th number in the sequence is defined to be the sum of the previous two, or formally
by this recurrence relation:

$$\begin{eqnarray}
\mathrm{Fib}(0) &=& 1 \\
\mathrm{Fib}(1) &=& 1 \\
\mathrm{Fib}(n) &=& \mathrm{Fib}(n - 1) + \mathrm{Fib}(n - 2)
\end{eqnarray}$$

I've chosen to start the sequence at index 0 rather than the more usual 1.

### Computing Fibonacci numbers

There's a few different reasonably well-known ways of computing the sequence. The obvious recursive implementation is slow:

{% highlight python %}
	def fib_recursive(n):
    	if n < 2: return 1
    	return fib_recursive(n - 1) + fib_recursive(n - 2)
{% endhighlight %}

An iterative implementation works in $$O(n)$$ operations:

{% highlight python %}
	def fib_iter(n):
	    a, b = 1, 1
	    for _ in xrange(n):
	        a, b = a + b, a
	    return b
{% endhighlight %}

And a slightly less well-known matrix power implementation works in $$O(\mathrm{log}\ n)$$ operations.

{% highlight python %}
	def fib_matpow(n):
	    m = numpy.matrix('1 1 ; 1 0') ** n
	    return m.item(0)
{% endhighlight %}

The last method works by considering the `a` and `b` in `fib_iter`
as sequences, and noting that

$$
    \left( \begin{array}{c}
    a_{n+1} \\
    b_{n+1} \end{array} \right)
    =
    \left( \begin{array}{cc}
    1 & 1 \\
    1 & 0 \end{array} \right)
    \left( \begin{array}{c}
    a_n \\
    b_n \end{array} \right)
$$

From which follows

$$
    \left( \begin{array}{c}
    a_{n} \\
    b_{n} \end{array} \right)
    =
    \left( \begin{array}{cc}
    1 & 1 \\
    1 & 0 \end{array} \right) ^ n
    \left( \begin{array}{c}
    1 \\
    1 \end{array} \right)
$$

and so if $$m = \left( \begin{array}{cc}
    1 & 1 \\
    1 & 0 \end{array} \right) ^ n$$ then $$b_n = m_{11}$$ (noting that unlike Python, matrix indexes are usually 1-based).

It's $$O(\mathrm{log}\ n)$$ based on the assumption that numpy's matrix power does something like exponentation by squaring.

Another method is to find a closed form for the solution of the recurrence relation. This leads to the real-valued formula: $$\mathrm{Fib}(n) = (\phi^{n+1} - \psi^{n+1}) / \sqrt{5})$$ where $$\phi = (1 + \sqrt{5}) / 2$$ and $$\psi = (1 - \sqrt{5}) / 2$$. The practical flaw in this method is that it requires arbitrary precision real-valued arithmetic, but it works for small $$n$$.

{% highlight python %}
	def fib_phi(n):
		phi = (1 + math.sqrt(5)) / 2.0
		psi = (1 - math.sqrt(5)) / 2.0
		return int((phi ** (n+1) - psi ** (n+1)) / math.sqrt(5))
{% endhighlight %}


### Generating Functions

A generating function for an arbitrary sequence $$a_n$$ is the infinite sum $$\Sigma_n a_nx^n$$.
In the specific case of the Fibonacci numbers, that means $$\Sigma_n \mathrm{Fib}(n)x^n$$.
In words, it's an infinite power series, with the coefficient of $$x^n$$ being the $$n$$th
Fibonacci number.

If we call this function $$F(x)$$ and use the definition of $$\mathrm{Fib}$$, we get $$F(x) = x^2F(x) + xF(x) + 1$$.
That's just saying that the $$n$$th Fibonacci number is the sum of the previous two, and that the coefficient of
$$x^0$$ is 1.

The magic trick of the method of generating functions is that we can solve this equation for $$F$$ to get $$F = \frac{1}{1 - x - x^2}$$.
This assumes that the infinite sum is convergent, but we know the fibonacci numbers grow like $$\phi^n$$ and that geometric
sequences $$\Sigma_n a^n$$ converge if $$a<1$$, so we know that if $$\|x\| < 1/\phi \simeq 0.618$$ then the sequence converges.

### An integer formula

Now we're ready to start understanding the Python code.

If we evaluate $$F(x)$$ at a negative power of 10, we'll be able to read off the first few Fibonacci numbers
in the decimal expansion of the result. That's because $$F(10^{-n}) = \mathrm{Fib}(0) + \mathrm{Fib}(1)/10^n + \mathrm{Fib}(2)/10^{2n} + \mathrm{Fib}(3)/10^{3n} + ...$$.
Before the Fibonacci numbers get larger than $$10^n$$ and they start interfering with their neighbours, the numbers are clearly visible in the expansion.

For example, the numbers $$1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89$$ are visible here:

$$ F(10^{-3}) = \frac{1}{1 - 10^{-3} - 10^{-6}} = 1.001002003005008013021034055089... $$

Negative powers of 10 aren't going to be convenient to code, and instead we'll use $$1/K$$ for some integer $$K$$. Then:

$$ F(1/K) = \frac{1}{1 - 1/K - 1/K^2} = K^2 / (K^2 - K - 1) $$

We still have the problem that this requires real arithmetic, but we can avoid that by multiplying up by a large enough integer.
We'll use $$K^n$$ as the large integer. Then:

$$ K^n \cdot F(1/K) = K^{n+2} / (K^2 - K - 1) $$

This conveniently puts the digits (in base $$K$$) of $$F(1/K)$$  starting at the $$-n$$th power of $$K$$ just above the decimal point.
Now, if we pick $$K$$ large enough, truncate the division at the decimal point, and take the result modulo $$K$$, we can find the $$n$$th Fibonacci number.

How big does $$K$$ need to be? Well we already know that the Fibonacci numbers grow like $$\phi^n$$, and since $$\phi < 2$$, $$K = 2^{n+1}$$ should be enough
to avoid overflow at position $$n$$.

So!

$$ \mathrm{Fib}(n) \equiv \frac{(2^{n+1})^{n+2}}{(2^{n+1})^2 - 2^{n+1} - 1} \equiv \frac{2^{(n+1)(n+2)}}{2^{2n+2} - 2^{n+1} - 1} \mathrm{mod}\ 2^{n+1} $$

If we use left-shift notation that's available in python, where $$a << k = a \cdot 2^k$$ then we can write this as:

$$ \mathrm{Fib}(n) \equiv \frac{4 << n(3+n)}{(4 << 2n) - (2 << n) - 1} \mathrm{mod}\ (2 << n)  $$

Observing that $$\mathrm{mod}\ (2 << n)$$ can be expressed as the bitwise and (`&`) of $$(2 << n) - 1$$, we reconstruct our original Python program:

{% highlight python %}
    def fib(n):
        return (4 << n*(3+n)) // ((4 << 2*n) - (2 << n) - 1) & ((2 << n) - 1)
{% endhighlight %}

Although it's curious to find a non-iterative, closed-form solution, this isn't a practical method at all. We're doing integer arithmetic with integers of size $$O(n^2)$$ bits, and in fact, before performing the final bit-wise and, we've got an integer that is the first $$n$$ Fibonacci numbers concatenated together!
