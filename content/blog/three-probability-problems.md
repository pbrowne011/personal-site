+++
title = "Three probability problems"
date = 2025-07-03T00:00:00Z
draft = false
author = "Pat"
description = "How much do I really understand?"
tags = []
categories = []
images = []

# set X (formerly known as Twitter) card image

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
keywords = ["anki", "probability", "problems", "binomial"]

# if post is part of a blog series
series = []

# if you want to weight a particular post's ranking
# weight = 1

# generate table of contents
toc = false
+++

Yesterday, I [tried](2025/07/02/lognormal) (and failed) to derive both the
density function and the median of the lognormal distribution. I added these
facts to my Anki deck when reviewing an older formula sheet, but broke the first
rule of a good memory practice: do not learn if you do not understand.

There are many other skeletons in this closet (what of the gamma distribution,
when I don't remember much about the gamma function outside of
factorials?). Instead of rooting them all out, I gave myself a quick test this
afternoon. How much probability do I understand?

I selected three problems from William Feller's probability textbook, ["An
Introduction to Probability Theory and Its Applications", Volume
1](https://www.goodreads.com/book/show/2378167.An_Introduction_to_Probability_Theory_and_Its_Applications_Volume_1). They
were arbitrary, but all questions I could have answered somewhat easily when
studying the subject actively.

#### Question 1 (Chapter IV, "Combination of Events")

> 7. *Quadruples in a bridge hand*.  By a quadruple we shall understand four
>    cards of the same face value, so that a bridge hand of thirteen cards may
>    contain $0, 1, 2,$ or $3$ quadruples.  Calculate the corresponding
>    probabilities.

Things that went well:
- I understood the problem well. Card-counting problems are common when learning
  to count, so I may be evaluating how well that heuristic worked, rather than
  my ability to count combinations of a different event.
- I remembered the difference between permutations and combination (after a
  couple of moments), and how to count different events when order matters and
  when it does not.

Things that did not go well:
- Counting problems rely on attention to detail. I did not realize until the end
  (just before I looked at the answer) that there were $13$, not $12$ cards in a
  hand. This affects, for instance, how many ways one can get a hand of three
  quads: in addition to the three sets, there's an extra card to be accounted
  for.
- It took me a second to remember why there are $\binom{52}{13}$ possible hands
  when dealing a deck of cards to four people. I got stuck on why $52!$ is in
  the numerator - aren't there only $13$ cards being dealt? - but eventually
  remembered what was in the denominator as well ($39!$).

I got all answers to this question correct except $\Pr(X=1)$, [^1] where I
didn't pay attention to detail and only accounted for $12$ of the $13$
cards. I also decided to subtract the probabilities of $1$ quad and $2$ quads by
the higher-numbered probabilities, but can't remember if I discarded that before
peeking at the key. The correct answers (as written by Feller) are:
$$
\begin{aligned}
\text{Let } p^{-1}=\binom{52}{13}\text{.  Then} \\\\[1.5ex]
\Pr(X=3) &= 40 \cdot \binom{13}{3} \cdot p \\\\[1.5ex]
\Pr(X=2) &= \binom{44}{5} \cdot \binom{13}{2} \cdot p \\\\[1.5ex]
\Pr(X=1) &= \binom{48}{9} \cdot 13 \cdot p \\\\[1.5ex]
\Pr(X=0) &= 1 - \sum_{x=1}^{3}\\;\Pr(X=x)
\end{aligned}
$$

Overall, I didn't fare too well here. I need to relearn how to count (and
practice it much more). I have Anki cards for permutations and combinations,
but some card additions that might help include
- giving a count and asking me to explain it to myself
- giving a permutation and asking for its combination (and vice versa)
- giving a count and adding a twist (given probability of three quads, what's
  the probability of four triples?)

A better strategy would be to read through the chapters on combinatorics and
conditional probability in several passes, pulling out understanding from each
pass to build intuition. It will be a good sign when I don't question myself on
why the number of potential hands is $\binom{52}{13}$. Not sure what I'll do
yet, but I'll keep this in mind as I encounter more of this type of Anki card.

#### Question 2 (Chapter IX, "Random Variables")

> 17. *Birthdays*.  For a group of $n$ people find the expected number of days
>     of the year which are birthdays of exactly $k$ people.  (Assume $365$ days
>     and that all arrangements are equally probable.

*Ah, the [birthday paradox](https://en.wikipedia.org/wiki/Birthday_problem)* he
exclaimed! as I proceeded to blunder through this problem. Again relying on
pattern matching, I remembered that there was something unique in the birthday
paradox, but couldn't remember what that thing *was*. I also misunderstood the
problem, as the birthday paradox asks about the case where $k \geq 2$.

Reeling, I considered the binomial distribution: we're given variables with the
names $n$ and $k$, and there's a $p$ ($\frac{1}{365}$). Applying this intuition,
I came up with
$$
P(X=k)=p^k(1-p)^{n-k}
$$
however, as the astute might notice, this is not the probability mass function
of the binomial distribution. That is instead
$$
P(X=k)=\binom{n}{k}p^k(1-p)^{n-k}
$$
which includes an actual binomial term to account for the ways in which these
events can be ordered. Counting matters quite a bit in probability.

In the end the correct answer (again, courtesy of Feller) is
$$
\binom{n}{k}364^{n-k}365^{1-n}
$$

Evaluating what I had originally (and adding the binomial term),
$$
\begin{align}
&= \binom{n}{k}\left(\frac{1}{365}\right)^k\left(1-\frac{1}{365}\right)^{n-k} \\\
&= \binom{n}{k}\frac{1^k \cdot 364^{n-k}}{365^k \cdot 365^{n-k}} \\\
&= \binom{n}{k}364^{n-k} \cdot 365^{-n} \\\
\end{align}
$$
I'm missing $365$ in the numerator. I did not account for another part of this
problem: we are not calculating a probability, rather, 

> the *expected number of days* of the year...

(strike two from the first problem, attention to detail)

Since this is an expectation, we multiply each day by its probability and sum
the days up. In mathematical terms,
$$
\begin{align}
\mathbb{E}[X] &= \sum_{x=1}^{365}\\;1 \cdot \Pr(X=x) \\\\[1.2ex]
&= 365 \cdot \Pr(X=x)
\end{align}
$$
giving us the missing $365$ from above.

Overall, not great: making the same mistakes, relying on pattern recognition
rather than intuition, not understanding the question.

#### Question 3 (Chapter IX)

> 21. Find the covariance of the number of ones and sixes in *n* throws of a
>     die.

I did not give this problem a full effort. I saw the term *covariance*,
attempted to recall the formula from memory, and gave up when I could not.

This problem attacked another area of probability (properties of random
variables) that I studied but can no longer recall. It wasn't too hard to answer
this with the formula for covariance in front of me: $\mathbb{E}[XY] - \mu_x
\mu_y = 0 - \frac{n}{6\\; \cdot \\;6} = -\frac{n}{36}$ (the throws are assumed
to be independent). My unfamiliarity with covariance is once again not great.

### What have I learned? What do I understand?

While answering these questions, I felt like someone returning to a place they
hadn't been to in awhile. Though it's only been six months since my last class
in statistics, the gaps in my knowledge have grown and eaten through much of the
intuition I used to have. Twelve months ago these problems would have been much
easier than they were today.

This indicated the same fundamental issue: I have *learned*, but it is clear I
no longer *understand*. I can tell you that $\text{Var}[X] = \mathbb{E}[X^2] -
\mathbb{E}[X]^2$, but if asked follow-up questions, there would be significant
gaps in my knowledge that come up.

Where do I go from here? Do I attempt to relearn probability from the ground up,
or take my half-built crooked tower of knowledge with me as I go? Or is the
choice binary?

The key point is that I now have an *explicit* choice. I know where my knowledge
stands currently better than if I kept going through Anki cards sporadically and
haphazardly. As a result, I can be more intentional about my memory practice and
what I choose to learn in general. I'll consider tradeoffs with, e.g.,
technologies I need to know for work, but now I know better what
I... [know](https://en.wikipedia.org/wiki/Circle_of_competence).


[^1]: Aside: I am undecided on what to use for $\LaTeX$ probability
    notation. When writing, I often use $P$ and $\Pr$, but prefer
    $\mathbb{P}\text{r}$ (without the double line in the bend of the P). I
    haven't found a symbol for that P yet and would want to turn it into an
    Emacs/$\LaTeX$ macro as well if it's complex.
