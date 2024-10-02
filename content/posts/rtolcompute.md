+++
title = "Right to left computability"
date = 2024-10-02T00:00:00Z
draft = false
author = "Pat"
description = "Exploring parallelizable bit tricks"
tags = ["draft", "computability", "boolean-logic", "bit-tricks",
"hackers-delight"]
categories = []
images = []

# set slug if you want to change a page
slug = ""

# modify type of page (posts show up on front page)
# types can be found in mainSections
type = "post"

# if you want to display modification date
# lastmod = 2024-03-01T00:00:00Z

# redirect old URLs to new URL for post
aliases = []

# SEO keywords
keywords = []

# if post is part of a blog series
series = []

# if you want to weight a particular post's ranking
# weight = 1

# generate table of contents
toc = false
+++

### Manipulating bits

Recently I've been going through [Hacker's
Delight](https://en.wikipedia.org/wiki/Hacker's_Delight). It is a joy to become
a "bit-twiddler" and explore some of the counter intuitive properties of using
bit wise operations in places you wouldn't expect them.

Chapter 2 focuses on "Basics", and covers bit manipulation tricks that perform
specific operations. It assumes a 32-bit machine (and word size), with signed
values represented in two's complement form. [^1] Some tricks presented include:
- $x \\; \\& \\; (x - 1)$: turn off the rightmost bit in a word
- $x \\; \\& \\; (x + 1)$: turn off the trailing 1's in a word
- $\neg x \\; \\& \\; (x + 1)$: create a word with a single 1-bit at the position of the
  rightmost 0-bit
- $x \\; \\& \\; (-x)$: isolate the rightmost 1-bit, produce 0 if none

Note that these hacks are much more useful than they might appear. The first
trick can be used to great effect to count set bits and calculate powers of two,
as demonstrated in Brian Kernighan's algorithm[^2]:
```
unsigned int v; // count the number of bits set in v
unsigned int c; // c accumulates the total bits set in v
for (c = 0; v; c++)
{
  v &= v - 1; // clear the least significant bit set
}
```
These bit wise operations allow for implementations of functions that are
branch-free and do not require shift instructions, saving valuable CPU
cycles. Additionally, they allow for word-parallelism: if a bit's value only
depends on bits to its right, one can optimize by executing single instructions
on multiple data, or
[SIMDs](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data).

One condition of these optimizations is that they cannot depend on overflow. For
example, if performing a SIMD addition on multiple words, carries must be
irrelevant to the operation - if they are important, then higher-order word bits
depend on lower-order word operations and carry bits, and the addition becomes a
sequential computation. Another condition is that the machine must use two's
complement arithmetic.

### Moving right to left

In 1977, Henry Warren (the author of Hacker's Delight) [published a short
paper](https://dl.acm.org/doi/abs/10.1145/359605.359632) in an attempt to
generalize these sorts of bit twiddling tricks. He proved a theorem on whether
or not one can use a coding trick of this class for a given function that they
want to optimize. The theorem, as stated in the book (pg. 13), is:

> THEOREM. A function mapping words to words can be implemented with
> word-parallel add, subtract, and, or, and not instructions if and only if
> each bit of the result depends only on bits at and to the right of each input
> operand.

This introduces the concept of **right-to-left computability**: if output bits
depend only input operand bits that are located either in the same position or
to the right (lower order bits), the function can be optimized with these bit
tricks by rewriting it in terms of the five functions that are mentioned,
providing the benefits described above.

An example function is then provided to demonstrate what this means. Consider
the following operations to compute the third bit of the output ($r\_2$):

$$r\_2 = x\_2 \mid (x\_0 \\; \\& \\; y\_1)$$

From this, it is clear to see that the output bit $r\_2$ does not rely on any
input bit $x\_i$ or $y\_i$ where $i > 2$. Since it depends only on the bit wise
operations `AND` and `OR`, the function to calculate $r\_2$ is right-to-left
computable. Further examples that are right-to-left computable include
multiplication (a constant number of additions), as well as other Boolean
operations (`XOR`, `NAND`, etc.).

An example of a function that is not right-to-left computable is something like
$\operatorname{signum}$.

$$
\operatorname{signum}(x) = \left\lbrace\begin{array}{rcl}
-1, & x < 0 \\\
0, & x = 0 \\\
1, & x > 0 \\\
\end{array}\right.
$$

$\operatorname{signum}$ takes a word as input and outputs the sign of that
word: $1$ if positive, $-1$ if negative, and $0$ otherwise. Almost all bits of
the output depend on the leftmost bit of the input. (The exception is the
leftmost bit $r\_0$, which depends only on itself - it is $1$ for both `0x1` and
`0xff..ff`, and $0$ for `0x0`. The next bit $r\_1$ depends on the leftmost bit.)

Another function described in the book is `>>`, right shift: you can't determine
an output bit $r\_i$ of a right shift $n$ places without knowing the input bit
to the left of it $(x\_{i+n})$.

### A fun problem

The book mentions several other functions that are not right-to-left computable
without further explanation. One of them is left shift by a variable
amount. The intuition is that the variable amount depends on either the whole
input or some subset of it, which necessarily includes bits to the right of your
output bit. Exercise 2 for the chapter challenges this claim. Courtesy of
Donald Knuth[^3]:

$$ x \<\< (x \\; \\& \\; 1) $$

This expression is equivalent to:

$$ x + (x \\; \\& \\; (-(x \\; \\& \\; 1))) $$

The question is: how is this possible? We have an apparent contradiction. Left
shift by a variable amount is not right-to-left computable, but above we have a
variable left shift expressed in terms of `AND`, `-`, and `+`, which implies
that it is right-to-left computable.

Consider the amount we are shifting left by. We shift left by either 0 or 1
bits, depending on the value of $x\_0$. Hence, the value of each output bit
$r\_i$ depends on $x\_0$, $x\_i$, and $x\_{i-1}$. We are shifting left by a
variable amount, but that amount depends on the rightmost bit $x\_0$, not any
other bit in the input.

On the other hand, the expression $x \<\< (x \\; \\& \\; 2)$ is not expressible
as a right-to-left computation, as the output bit $r\_0$ depends on the input
bit $r\_1$.

### Infinite-bit registers

Richard Allen has a blog post, [Right to Left
Computability](https://rsaxvc.net/blog/2020/11/21/Right_to_Left_Computability.html)
(archived
[here](https://web.archive.org/web/20241002103622/https://rsaxvc.net/blog/2020/11/21/Right_to_Left_Computability.html)),
that explores this theorem further. In the latter half of the post ("A
hypothetical machine"), he explores a hypothetical infinite-bit
machine. The machine has registers and instructions. The registers are capable
of storing an infinite number of bits. The instructions are all right-to-left
computable, and are based on the five instructions allowed by Warren: `AND`,
`OR`, `NOT`, `+`, and `-`.

The construction, since it is not a two's complement machine (infinite
bit strings), is subtly different from the one used as the basis of Hacker's
Delight. For instance, what happens when one performs `0 - 1`? In this hypothetical
world, since we are dealing with infinite bit string registers, it makes sense to
consider what happens when we decrement a number. To perform this operation
right-to-left, you flip each zero bit you find until you get to a non-zero bit,
which you make zero.

In two's complement format, every bit is flipped. On a 32-bit register, this
results in the word `0xffffffff`, which is $-1$ in two's complement form.  With
infinite registers, the same applies, assuming we are able to flip all of the
infinite bits in `0`. However, you will never know the sign of a number, as
you'll never be able to read its last bit (it is infinite).

### Turing completeness

However, I had trouble understanding why such a machine was not Turing complete,
as the author claimed. To be a Turing-complete machine, this hypothetical
infinite-bit machine must be able to emulate an arbitrary Turing machine. As
stated
[here](https://www.cs.odu.edu/~zeil/cs390/latest/Public/turing-complete/index.html)
and
[elsewhere](https://langdev.stackexchange.com/questions/32/how-can-i-check-if-a-new-language-is-turing-complete),
a heuristic for Turing completeness is to try to implement `if` and `while`
loops with your language. For this hypothetical machine, we require some form of
branching to implement this.

General branching is not possible with infinite registers. Consider (as Allen
does) the instruction branch-if-greater-than (`JA` on x86). This requires a
comparison of two operands to determine which is greater. With positive values,
this is feasible, but what about negative values? Here, one would have to read
the entire value of the negative register, which is not feasible. What about
infinitely repeating values? Again, there is no obvious solution.

Two that come to mind for me are 1) fixed size comparison and conditional
branching instructions and 2) secondary registers to track sign. The first is
dismissed in the blog post as "a pretty big hack". Hypothetically, it could
limit what the language recognizes as valid, but it allows for branching in some
form, which permits Turing completeness (conjectured without proof here). The
second solution works in the case of "rational" registers, but not in the case
of infinitely repeating registers.

In either case, it is interesting to consider how deviating from the setup of
Hacker's Delight changes the model of computation. Many of the tricks I've read
so far rely on either fixed word size or two's complement representation (or
both) to be effective, which makes sense today given these are defaults in
modern computers.


[^1]: For resources on two's complement, there are many - search the Internet
    far and wide! Three that I've found helpful are [Thomas Finley's
    introduction](https://www.cs.cornell.edu/~tomf/notes/cps104/twoscomp.html),
    [Evan Martin's quick
    note](https://neugierig.org/software/blog/2023/06/twos-complement.html), and
    JF Bastien's [standards proposal for
    C++20](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0907r0.html).

[^2]: [See the original
    source](https://graphics.stanford.edu/~seander/bithacks.html#CountBitsSetKernighan)
    for the code, details, and other implementations of counting bits. Sean
    Anderson's page is another wonderful resource for bit twiddling. Note that
    Knuth points out that the method was first published in 1960 by Peter
    Wegner, and later (independently) by Derrick Lehmer - before K&R used it in
    the second edition of "The C Programming Language" in 1988.

[^3]: Knuth has a habit of finding his way into many textbooks and stories. To
    some degree, "all roads lead to Knuth" when it comes to computer
    science. Some recent examples off the top of my head:
    - In the book "A Book on C", by Kelley and Pohl, Knuth suggests to the
      authors an algorithm (exercise 1.15) that presents a better way to compute
      an average of a sum. Sadly, the exercise does not ask to explain why this
      method is better, only why it is correct. For the explanation, [read this
      post](https://nullbuffer.com/articles/welford_algorithm.html).
    - In [an interview on
      CoReview](https://corecursive.com/066-sqlite-with-richard-hipp/#b-trees-and-the-art-of-computer-programming)
      from 2021, Richard Hipp mentions that when he implemented B trees in
      SQLite, he looked to Knuth's TAOCP to learn and implement the
      algorithm. To use the algorithm for deleting a B tree, he had to solve the
      exercise at the end of the chapter (sourced from [this HN
      comment](https://news.ycombinator.com/item?id=38444870)).
    - When talking with one of my cryptography professors in office hours, he
      mentioned looking for an algorithm he needed to prove something in his
      PhD thesis. He asked around, but eventually went to TAOCP... where, sure
      enough, Knuth had presented details on the algorithm and its analysis.
    
