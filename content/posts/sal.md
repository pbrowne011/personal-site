+++
title = "SAL/SAR/SHL/SHR — Shift"
date = 2024-11-04T00:00:00Z
draft = false
author = "Pat"
description = "The shift operation has a maximum operand size"
tags = ["C"]
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
keywords = ["x86", "asm", "SAL", "shift left", "bitwise shift"]

# if post is part of a blog series
series = []

# if you want to weight a particular post's ranking
# weight = 1

# generate table of contents
toc = false
+++

I've been working through some of the exercises in ["The C Programming
Language"](https://en.wikipedia.org/wiki/The_C_Programming_Language) by
Kernighan and Ritchie, and ran into an issue in Chapter 2 ("Types, Operators,
and Expressions").

**Exercise 2-7**:

> Write a function `invert(x,p,n)` that returns `x` with the `n` bits that begin
> at position `p` inverted (i.e., 1 changed into 0 and vice versa), leaving the
> others unchanged.

This seemed easy enough, and I wrote the following function as a solution: [^1]

```C
/* invert.c */

unsigned int invert(unsigned x, int p, int n)
{
    unsigned mask = ~(~0 << n) << p-(n-1);
    return x ^ mask;
}
```

Earlier, the authors wrote a similar function `get_bits(x,p,n)`, and stated
some assumptions they made when writing it:

> We assume that bit position 0 is at the right end and that `n` and `p` are
> sensible positive values.

Originally, I thought "sensible positive values" implied maximum and minimum
bounds on `p` and `n`:
- **`n > 0`, and `p >= 0`**: it doesn't make sense to have `n=0` (what's the
  point of inverting 0 bits?), and setting `p` or `n` to a negative value has no
  meaning in the context of absolute position or size.
- **`p >= n-1`**: inverting more bits than are
  available on the right side of a number doesn't make sense either.

So I added some error checking:
```c
unsigned int invert(unsigned x, int p, int n)
{
    if (p < 0) {
        fprintf(stderr,
                "Error: position (p=%d) less than 0\n",
                p);
        return 0;
    } else if (n < 1) {
        fprintf(stderr,
                "Error: size (%d) is not a positive number\n",
                n);
        return 0;
    } else if (p < n-1) {
        fprintf(stderr,
                "Error: position (%d) less than n - 1 (%d)\n",
                p, n-1);
        return 0;
    }
    
    unsigned mask = ~(~0 << n) << p-(n-1);
    return x ^ mask;
}
```

I set out to write some test cases, starting with several values for `x` and
three sets of cases for `n` and `p`:
- `n = 32, p = 31`
- `n = 16, p = 31`
- `n = 16, p = 15`

These correspond to inverting all four bytes, the first two bytes, and the last
two bytes of `x`, respectively. I ran the program and printed the relevant
values to stdout:

```
x:                  00000000000000000000000000000000
~x:                 11111111111111111111111111111111
invert(x, 31, 32):  00000000000000000000000000000000
```

Huh? The function call is to invert the last 32 bits of `x`, starting from
position 31 (the first position). `~x` should be returned here. Instead, the
function is returning `x`.

However, later test cases print the correct output to the terminal:

```
x:                  00000000000000000000000000000000
~x:                 11111111111111111111111111111111
invert(x, 31, 16):  11111111111111110000000000000000
```

I ran the program in `gdb` and found an error with this expression:

```c
    ~(~0 << n) << p-(n-1)
```

The mask value was being set incorrectly in the first case (`p=31,n=32`):
instead of masking all of the bits (`0xffffffff`), it was being set to
`0x0`. The output of the function was `x ^ 0 = x`.

I went to the assembly code to see what was being generated (written below with
explicit values for `p` and `n`):

```x86asmatt
# invert.s
    ...
invert:
    ...
	movl	$31, -12(%rbp)
	movl	$32, -8(%rbp)
	movl	-8(%rbp), %eax
	movl	$-1, %edx
	movl	%eax, %ecx
	sall	%cl, %edx
    ...
```

Other than not being optimized, it looked fine. The issue was that when I ran
`sall %cl, %edx`, the destination register wasn't being shifted. What was I
doing wrong?

I then asked Claude 3.5 Sonnet for input on what was going on, and it responded
with the following (among other comments):

> According to the C standard (6.5.7), shifts by an amount equal to or greater
> than the width of the type are undefined behavior. 

Interesting... but why was this defined this way in the C standard? I looked at
the C standard ([ISO/IEC 9899:2017, aka the final draft of
C17](https://web.archive.org/web/20181230041359if_/http://www.open-std.org/jtc1/sc22/wg14/www/abq/c17_updated_proposed_fdis.pdf)),
and found that for a left shift:

> 6.5.7

> The integer promotions are performed on each of the operands. The type of the
> result is that of the promoted left operand. If the value of the right operand
> is negative or is greater than or equal to the width of the promoted left
> operand, the behavior is undefined.

So, Claude was correct and the behavior is undefined. I'm using a laptop with an
Intel processor - what is the defined behavior within Intel's ISA?

According to the Intel Developer Manual, pg. 4-593:

> The destination operand can be a register or a memory location. The count
> operand can be an immediate value or the CL register. **The count is masked to
> 5 bits** (or 6 bits with a 64-bit operand). The count range is limited to 0 to
> 31 (or 63 with a 64-bit operand). A special opcode encoding is provided for a
> count of 1.

In the code above, `n` is defined as `unsigned`, which means that it has type
`int`. On my machine, `sizeof(int) = 32 bits`. So in the first test case when `n
= 32`, the entire number is being masked off (`0x20 ^ 0x1f == 0x0`). However, in
the other two test cases, `n = 16`, which is within the lower five bits of the
number. Mystery solved. [^2] [^3]

It seems that there is more to "sensible positive values" than I first
realized. I thought of minimum bounds on both `p` and `n`, as well as a maximum
bound on `p` - but no maximum bound on `n`.

The good news is that, with two lines of code to debug, this was [an easy
bug](https://www.cs.princeton.edu/~bwk/tpop.webpage/debugging.html) to
find.

## An overflow-safe approach

A major problem with C is that there is no mechanism to catch this undefined
behavior at runtime. Running `gcc` with`-Wall` does provide a warning when the
second operand is a constant:

```
invert.c:71:26: warning: left shift count >= width of type [-Wshift-count-overflow]
   71 |     unsigned mask = ~(~0 << 32) << p-(n-1);
      |                          ^~
```

However, when I run the program, there are no errors (hence my debugging
adventure). In C, **undefined behavior silently produces implementation-defined
results**, so you have to be extremely careful and aware of what is undefined in
the C standard when writing code.

One of Rust's main features is to avoid undefined behavior like this in C. It
can detect memory errors, overflows, and has runtime safety checks in debug
builds.

Here is an implementation of `invert(x,p,n)` in Rust:

```rust
/* invert.rs */

fn invert(x: u32, p: i32, n: i32) -> u32 {
    let mask = !(!0_u32 << n) << (p - (n - 1));
    x ^ mask
}

fn main() {
    let result = invert(0xFFFFFFFF, 31, 32);
    println!("result: {:032b}", result);
}
```

The code produced no compilation errors, but on runtime, the integer safety
checks catch this error:

```
$ ./insert
thread 'main' panicked at insert.rs:2:17:
attempt to shift left with overflow
```

This runtime check (in debug builds) catches overflow attempts regardless of
whether the shift amount is known at compile time or computed from variables.

I wrote a more direct implementation to see if `rustc` could detect a hardcoded
left shift outside of the allowed bounds (like `gcc`):

```rust
/* insert_const.rs */

fn main() {
    let x: u32 = 0xFFFFFFFF;
    let p: i32 = 31;
    let n: i32 = 32;
    
    let mask = !(!0_u32 << n) << (p - (n - 1));
    let result = x ^ mask;
    
    println!("x: {:032b}", x);
    println!("mask: {:032b}", mask);
    println!("result: {:032b}", result);
}
```

And it did:

```
$ rustc insert_const.rs
error: this arithmetic operation will overflow
 --> insert_const.rs:6:17
  |
6 |     let mask = !(!0_u32 << n) << (p - (n - 1));
  |                 ^^^^^^^^^^^^^ attempt to shift left by `32_i32`, which would overflow
  |
  = note: `#[deny(arithmetic_overflow)]` on by default

error: aborting due to 1 previous error
```

This is one small example in the larger picure of how Rust and C prioritize
different features: C allows for direct hardware access and flexibility, while
Rust mandates strict safety requirements.


[^1]: Ironically, I did not come up with the idea for just XORing the mask with
    the input until I read the assembly output by `gcc` while debugging the
    function.

[^2]: After going through all of this, I also looked in Appendix A of TCPL to
    see if the authors mention this behavior. They do, on pg. 206:
    
    > The shift operators `<<` and `>>` group left-to-right. For both operators,
    > each operand must be integral, and is subject to the integral
    > promotions. The type of the result is that of the promoted left
    > operand. **The result is undefined** if the right operand is negative,
    > **or greater than or equal to the number of bits in the left expression's
    > type**. 
    
[^3]: I also checked the ARM reference manual for 32-bit systems for their
    implementation of logical shift left (`LSL`). It operates similar to
    the Intel x86 instruction - it has a valid range of values (0 to 31,
    inclusive), but does not define behavior if the operand is outside of this
    range. I think this might be left up to the chip designer to implement, but
    will test this on the ARM Cortex-M4 at some point.
