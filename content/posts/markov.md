+++
title = "Real-life examples of Markov chains"
date = 2024-03-04T00:00:00Z
draft = false
author = "Pat"
description = "Trying to model daily events in my life using Markov chains"
tags = ["markov-chain", "probability"]
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
keywords = ["markov chains", "probability", "law of total probability"]

# if post is part of a blog series
series = []

# if you want to weight a particular post's ranking
# weight = 1

# generate table of contents
toc = false
+++

One of the classes that I'm taking this semester is on
[stochastic processes](https://www.bu.edu/academics/cas/courses/cas-ma-583/).
Stochastic processes are sequences of random variables $X_1, X_2, ..., X_n$,
where $n$ represents a specific moment in time.[^1] The first topic we've
focused on this semester has been
[Markov chains](https://en.wikipedia.org/wiki/Markov_chain), specific
stochastic models that only depend on the current state. They are unique in
that future events are only affected by where you currently are, not where you
have been in the past. Markov chains arise from the 
[law of total probability](https://statproofbook.github.io/P/prob-tot.html),
which is a way to express the probability of an event as the sum of many other
probabilities. In general, when dealing with a stochastic process, you have
to condition the probability of an event on all previous events before, as they
all have some effect on whether or not you reach that state. Mathematically,
this can be expressed as

$$P[X_{n+1} = m \mid X_0 = i, X_1 = j, ..., X_n = k]$$

However, with Markov chains, we only need to focus on the most recent event.

$$P[X_{n+1} = m \mid X_n = k]$$

This is a remarkable simplification. We are able to ignore all previous events
and condition our probability on $X_n$. This greatly simplifies both the
conceptual understanding of each probability and the number of calculations
required to obtain it.

Markov chains appear everywhere: the Wikipedia page for Markov chains lists
several general examples, including random walks, board games played with dice
(assuming players have no agency), weather predictions, and the stock
market.[^2] Other examples we've gone over in class include bacteria
reproduction, the classic PageRank algorithm. 
However, I've become curious: what are some nonobvious things in my life
specifically that I could potentially model with Markov chains?

Some qualifiers first: I will only be focused on chains with discrete states
that I can quantify (*Snakes & Ladders* has 100 squares, gamblers have $N$
dollars, etc.). I want to pick things where I can answer a meaningful question
about them. Some examples of these questions could include:

- How long does it take me to do this?
- What is the probability that I end up in state $q$ at some point? What about
the probability that I do so before time $t$?
- Is there some absorbing state that I will always go to?

Below are some events I came up with:
- **Will I go for a run this morning?** I like going for runs in the morning,
and this is usually dependent on whether I ran the day before or not. I am more
likely to run if I hadn't run the day before, and vice versa, as I like to take
rest days and alternate. This is expressable as a Markov chain with two states:
either I ran or I didn't. To make it more concrete, simple Markov chains are
often expressed in terms of their transition probability matrix. We can make
an example transition matrix, loosely estimated based on my running data from
last year:
$$
\begin{array}{c c} 
& \begin{array}{c c} 0 & 1 \\ \end{array} \\
\begin{array}{c c}0\\1 \end{array} &
\left[
\begin{array}{c c}
0.05 & 0.95 \\
0.61 & 0.39 
\end{array}
\right]
\end{array}
$$
For those less familiar with Markov chains, there are a few things to note about
this matrix. Each index represents a state. The indices on the left are my state
from yesterday, and the indices on top are my state today. If I ran yesterday,
the probability that I don't run today at time $n+1$ is
$$P[X_{n+1} = 0 \mid X_n = 1] = \left(P^n\right)_{ij} = P_{10} = 0.61$$
If I ran yesterday but don't run today, I am transitioning from state 1 (the
left index) to state 0 (the top index). We can look this up in the matrix and
obtain this probability. This is especially useful when we want to predict many
steps in the future, as we can just raise the matrix $P$ to the $n$th power and
look up the transition probability based on our initial state. The power of
Markov chains lies in turning complex probability calculations into simpler
linear algebra problems.[^3]
- **What building will I study in this afternoon?** The building I will study
in on a given day is often based on where I studied the day before. I have
three building that I like: the library, the data science building, and the
general science building. I'll often choose a building different from the one
I studied in the day before, and so my one-step transition matrix might
look something like:
$$P = 
\begin{bmatrix}
0.2 & 0.4 & 0.4 \\
0.5 & 0 & 0.5 \\
0.3 & 0.6 & 0.1
\end{bmatrix}
$$
where state 0 is the library, state 1 is the data science building, and state 2
is the general science building.
- **What am I going to eat for dinner tonight?** Again, this is another decision
I'll make that is based on what I did the night before. If I cooked rice and
beans yesterday, I'll often have leftovers the next day; howver, if I ate ramen,
I'll try to make something different. I could express my states as 
the different meals that I eat (rice and beans, spaghetti, and ramen), and set
up a matrix similarly to how I did in the last two examples.
- **What will I choose to study tonight?** This is similar to where I will
choose to study because often the subject of the day is oen where I have an
assignment due. It's as if I'm putting out a fire in that subject today; as a
result, I am less likely to study the same subject two days in a row, and more
likely to move to another one to put out a fire there. Furthermore, my
assignments for most of my classes follow a weekly schedule: one class will
have them due on Mondays, another on Thursdays, etc. As a result, each row
in my transition matrix might have one probability close to 0 (the probability
that I study the same thing twice in a row), one probability close to 1 (the
class with an assignment due the next day) and the rest as a small nonzero
number.
- **Do my friends want to go gambling?** This is a two-state Markov chain, like
my first example. For some reason, two of my good friends from high school want
to go to the casino when we hang out instead of, say, someone's house, or a
restaurant or sports event. However, how much they want to go to the casino is
dependent on their most recent experience. If they won, they are more likely to
want to go back immediately. On the other hand, if the memory of a recent loss
is fresh on their mind, they might suggest another activity instead.

After going through all of these examples, one might see what they have
in common: human behavior. I am a human, and though I could model any one of
these processes with a Markov chain, some would fit better than others (and
none would be perfect). Just to pick apart two examples, what I eat for dinner
is affected by more than what I ate the night before: it depends on what food
I have left in the fridge, where I am eating dinner, who I am eating with, if
I've eaten lunch, etc. Additionally, what I choose to study might be more
directly affected by what is due soon than what I studied the day before. If I
have a test in a class the next day, I'll be more focused on that than the class
I'm ususally doing homework for that day.

This is why Markov chains are often more useful when there are less factors
affecting the outcome of an event. If this is the case, it is more likely that
the previous event is the determining factor. The closer a situation is to a
board game relying on the outcomes of a dice roll, the better it can be modeled
by these types of Markov chains. There are other Markov processes, such as
[hidden Markov models](https://en.wikipedia.org/wiki/Hidden_Markov_model), that
are modeled based on different assumptions (for example - what if you can't 
observe the states of the process directly?).

Note also that some questions one might answer with Markov chains in general are
not useful here. Rarely do I have an activity where I reach an absorbing state,
one where I will not move to some other state. The probability that I end up
in a particular state at some point is not as useful as the probability that
I do so before a certain time. Also, the expected time to complete some process
is not useful for any of the examples I have picked. When will I "finish"
running, or eating dinner?

Though Markov chains are models (not exact calculations), they can prove very
useful in other situations, such as 
[applications of random walks](https://en.wikipedia.org/wiki/Random_walk#Applications),
[modeling Brownian motion](https://en.wikipedia.org/wiki/Brownian_motion),
and [many other areas](https://en.wikipedia.org/wiki/Markov_chain#Applications).
However, it is important not to forget that they are a tool that can be applied
to whatever we choose, depending on what process we are trying to mode. Even
when trying to figure out how likely I am to eat ramen tomorrow.


[^1]: One way to understand what the term *stochastic process* refers to is
to break it down word by word. If something is *stochastic*, it means that it
is inherently random, and that it can be described by a probability 
distribution. A *process* occurs over time. Putting those together, you get a
sequence of events that are time-dependent, where each event can be modeled by
some probability distribution (and represented as a random variable). Because
each event is time-dependent, future states can be dependent on current and
past states, allowing one to model the whole system.

[^2]: See [here](https://en.wikipedia.org/wiki/Examples_of_Markov_chains)

[^3]: Note how each row of the transition matrix sums to 1. This is because of
the law of total probability: the probability that event $X_n = j$ occurs is
equal to the sum of probabilities that partition the set. That is,
$P[X_n = j] = \sum_i P[X_n + 1 = i \mid X_n = j]$. Since we are already in 
state $X_n = j$, $P[X_n = j] = 1$, and
$P[X_n + 1 = i \mid X_n = j] = P[X_n + 1 = i]$.
