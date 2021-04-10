---
title: "My Infinity is Bigger Than Yours"
date: 2019-12-15T13:51:22-08:00
draft: false
math: true
---

## What is Infinity?

We are going to start this post by trying to figure out what infinity is. What
do you think of when you think of infinity? Most people know that it's this
uncountable number or thing (or is it...?). There is no beginning or end, it
just goes on forever.

$\infty$ is defined slightly differently based on what field of math its used
in. For example, in calculus (or real analysis), infinity is used to define an
unbounded limit. Far more interesting is how infinity is used in set theory.
Georg Cantor created set theory and was fascinated by the idea of infinite
sets, such as the set of all natural numbers, or the set of real numbers. He
had this notion that there are different "sizes" of infinity, that different
sets can be infinite, but there are sets that are bigger than others even
amongst these infinite sets.

Cantor realized that there are different infinite sets with different
cardinalities. Some infinite sets are bigger than others, but how could that
be? I mean, they're both *infinite*, there is no actual "count," so to speak.
How could you possibly begin to say that one infinity is bigger than another?

## A Little Bit of Set Theory

In order to explain how we can have different sizes of infinity, we need to
formalize some mathematical definitions concerning sets, functions, and
cardinality.

### Set and Functions

Most people are familiar with the basics of functions, so a lot of this
shouldn't be new information. I just want to clear up a little bit of notation.

Let's suppose we have a function $f$ that provides some sort of relationship
or mapping from the set $X$ to the set $Y$. We can denote this as $f: X
\to Y$. We call $X$ the *domain* of the function, and $Y$ the *codomain*.
The codomain is the set of outputs a function can *possibly* have. Now, if we
know what the input set actually is, we can determine the *image* of the
function, which is simply the set of the outputs that the function actually
produces.

As an example, I will define $f$ as $f(x) = 2x$, and say that $f: R \to
R$. The function $f$, in this example, is a function that provides a mapping
from the set of real numbers to the set of real numbers. Now suppose that I
have a set of inputs I am using: $i = \{0, 1, 2\}$. The image (also known as
range) is $r = \{0, 2, 4\}$, because those are the values that the function
actually produced.  The codomain, just to reiterate, is the set of outputs that
the function *can* produce, whether or not we actually produce all of them.

In set theory, there are three types of functions that are critical to
understanding cardinality and the concept of infinite sets:

- injective functions (1-1 mappings)
- surjective functions (onto mappings)
- bijective functions

#### Injective Functions

We define a function $f: X \to Y$ as injective if and only if:

$$
\forall a, b \in X, f(a) = f(b) \implies a = b
$$

What this is really saying is that the function has to have a unique mapping
between the set $X$ and set $Y$. For every element in $X$, there is a
unique element in $Y$, so the only time that $f(x) = f(y)$ is if $x = y$.
If there is ever a case where different input values that lead to the same
output value, the function is not an injective function. For example, if we
define $f(x) = x^2$, we can trivially prove that it's not injective.  $f(-2)
= f(2)$, but $-2 \neq 2$. This fails the test for injective functions
because it is not a unique mapping between $X$ and $Y$. The uniqueness
factor also makes it clear why bijective functions are often referred to as a
1-1 correspondence or a 1-1 mapping.

#### Surjective Functions

A function $f: X \to Y$ is surjective if and only if:

$$
\forall y \in Y, \exists \in X, f(x) = y
$$

To break this down a little more, we have this set $Y$, which represents
every possible output from the function $f$. According to this definition,
a function is surjective if for every value in the set $Y$, there exists some
value in $X$ such that $f(x) = y$. Really, we're just saying that for every
element in the output set, there needs to be *some* value in the input set that
can produce the output.

Let's go back to having $f(x) = x^2$. Is this surjective? I don't know,
because we don't know what the mapping is between sets. If we define $f: Z \to
N$ ($f$ is a function from the integers to natural numbers), then yes! It is
surjective, because for every natural number, there is some integer where
$f(x) = y$. Also, remember that it doesn't have to be a 1-1 mapping like with
injective functions. If we defined $f: Z \to Z$ (a mapping from the natural
numbers to the natural numbers), then $f$ is no longer surjective, as it
can't produce any negative integers, and does not hit every element in the
codomain.

The easy way to summarize this is that for every element in $Y$, there's
*some* element in $X$ that can produce it given our function, and it doesn't
necessarily have to be unique.

#### Bijective Functions

This part is easy, a bijective function must meet the criteria for both
injective and surjective functions.

### Cardinality

You may be wondering what these different types of functions have to do with
cardinality. Well, the way we compare the cardinalities of different sets is by
proving that functions that meet certain criteria exist between these sets.
Given two sets: $X$ and $Y$, we have three cases (these can be trivially
flipped for the greater than and greater than or equal to cases).

* $|X| = |Y|$
* $|X| \leq |Y|$
* $|X| < |Y|$

In order to prove that $|X| = |Y|$, we need to define a bijection between
$X$ and $Y$. Establishing the bijection means that every element in $Y$
has only one corresponding element in $X$ and vice versa, which means that
they have to have the same number of elements, which is why we can say their
cardinalities are equal.

To say that $|X| \leq |Y|$, we define an injection from $X
\to Y$. This says that for every element in $X$, there is a unique
corresponding element in $Y$, but it's not necessarily that case that we can
produce every element in $Y$, so in effect we are proving that $X$
corresponds to a subset of $Y$.

In order to say that $|X| < |Y|$ (a strict less than, which is
different from less than or equal to), we say that there is an injection from
$X$ to $Y$, but no surjection, which means that while we have unique
mappings between $X$ and $Y$, we know that we can't cover every element in
$Y$, so $X$ has to correspond to a strict subset of $Y$.

### Infinite Cardinality

It's pretty easy to reason about proving cardinality relationships between
sets, but it turns out that you can do it for infinite sets too. Let's take the
set of natural numbers (these are the integers that are greater than or equal
to zero). It's an infinite set of numbers, and I'm going to say that the set of
even natural numbers has the same cardinality as the set of natural numbers.
They are infinite, but have the same size. How would we go about proving this?

Let's say the set of natural numbers is a set called $N$, and the set of even
natural numbers is called $E$. I'm going to define a function $f: N \to E$
such that $f(n) = 2n$. It's pretty easy to see that it's surjective, every
even number can be produced by doubling an odd number, regardless of how big it
is, so we know it works and will extend infinitely. We also know it's
injective, because for every unique natural number, the doubling operation will
provide a set of unique even numbers. If you doubt me, try to think of a
counterexample. Is there any even natural number that can't be produced by
doubling a natural number? Is there any case where doubling two different
natural numbers will produce the same even number? It's almost unbelievable
because it is so simple, but that's pretty much how it goes. The basic idea is
intuitive: we say that two sets have the same size if you can basically
"assign" every element from one set to an element to the other set. If we have
a sound pairing between elements from both sets, they *must* have the same
number of elements.

In my opinion, this is a boring result. Two infinite sets that have the same
cardinality? I think the opposite is far more interesting: sets that are
infinite that *don't* have the same cardinality.

We are going to use a technique by Georg Cantor is called the "diagonal
argument" to show that there exist sets that do not have a 1-1 correspondence
with $N$ (the set of natural numbers).

We start by constructing a set that we will call $B$ that consists of every
possible sequence of infinite binary sequences and is bijective with $N$. The
set $B$ is an enumeration over all of them.

{{<katex display>}}
\begin{gather*}
b_1 = \{ 0, 0, 0, 0, \cdots \} \\
b_2 = \{ 1, 0, 0, 0, \cdots \} \\
b_3 = \{ 1, 1, 0, 0, \cdots \} \\
b_4 = \{ 1, 1, 1, 0, \cdots \}
\end{gather*}
{{</katex>}}

Given this sequence, we are going to get a number from the diagonals in the
sequence. We are going to take the $n^{th}$ element from $b_n$ and make
that the $n^{th}$ digit of the new number (the numbers are bolded).

{{<katex display>}}
\begin{gather*}
b_1 = \{ \textbf{0}, 0, 0, 0, \cdots \} \\
b_2 = \{ 1, \textbf{0}, 0, 0, \cdots \} \\
b_3 = \{ 1, 1, \textbf{0}, 0, \cdots \} \\
b_4 = \{ 1, 1, 1, \textbf{0}, \cdots \}
\end{gather*}
{{</katex>}}

So, our new number is $0000 \dots$. We will create a new number by taking the
binary complement of the number (we're just flipping the 0s and 1s), to create
$1111 \dots$.

Do you remember how I told you that that we constructed the set $B$ as an
enumeration of every possible binary sequence? If that's true then how could we
construct the new number? We know that this new number doesn't exist in the set
$B$ because every digit at each position in the new number has to differ from
the existing numbers in the set, because we specifically took the complement at
each position. If you recall, $B$ is the set of all infinite binary
sequences, and was constructed to have a binary sequence for every natural
number. We just identified a binary sequence that's not in $B$, which is
contradictory. If there exists an extra number, that means that $B$ has at
least one more number than the set of natural numbers, so we lose the bijection
between $B$ and $N$, meaning that $|B| > |N|$.

There are two types of infinite sets: sets that are *countably* infinite and
*uncountably* infinite sets. Countably infinite sets are bijective with the set
of natural numbers, whereas uncountably infinite sets cannot. We just proved
that uncountably infinite sets exist, and it turns out that infinity can be
countable too.
