---
title: "Lambda Calculus"
date: 2020-04-11T21:57:04-04:00
math: true
---

In 1936, Alonzo Church invented a universal model of computation called "lambda
calculus." This system expresses computation as reductions on *lambda
expressions*, which are basically just functions and variables. The system is
simple but incredibly expressive, and serves as the foundation for programming
languages such as [Haskell](https://www.haskell.org) and
[Idris](https://www.idris-lang.org).

The significance of having a universal model of computation is that it provides
a way of solving any problem that can be expressed in the system, and changes
problems from "How can I calculate this," to "Can I express this properly?"
Lambda calculus is also Turing complete, and even more impressively, was
invented in the 1930s independently of Turing.

I'm not going to proclaim to be an expert on category theory or lambda
calculus, but I would like to show you some of the basics, because I think that
seeing how simple it is to perform computations with lambda calculus will give
you some idea of how elegant this model is.

## Getting Started: Lambda Terms

In lambda calculus, we have three types of *lambda terms*: *expressions*,
*variables*, and *abstractions*. Expressions are variables, abstractions, or a
combination of the two.

### Variables

Variables are easy to reason about, they're just like the variables you
encounter in math or programming. We can define some variable \\(x\\), which
represents a potential value. We don't know what the value is, and in this
version of lambda calculus, *untyped* lambda calculus, \\(x\\) can be anything.

### Abstractions

Abstractions are basically just functions. In lambda calculus, an abstraction
is denoted by the \\(\lambda\\) symbol (hence, *lambda* calculus). I'm going to
show you the most basic abstraction possible, the identity function:

$$
\lambda x . x
$$

The section from the \\(\lambda\\) symbol to the \\(.\\) is called the *head*
of the abstraction. Everything after the \\(.\\) is called the *body*.

This function is called the identity function because it corresponds to:

$$
f(x) = x
$$

It simply returns the value it was given.

Let's break down the abstraction. First, we have the \\( \lambda \\) symbol,
which declares the abstraction. After the \\( \lambda \\), we see our first
variable, \\( x \\). The first \\( x \\) *binds* the variable inside the
abstraction. This means that all instances of \\( x \\) in the body of the
function correspond to the input that was given for \\( x \\). Any variables
you see in the head of a lambda abstraction bind the variables in the body of
the abstraction.

In even more concrete terms, let's rewrite the abstraction I declared as a
regular function:

$$
\lambda x_{a} . x_{b} \newline
f(x_{a}) = x_{b}
$$

I added the subscripts so I can clearly refer to which \\( x \\) I'm talking
about. For the purpose of the actual math, ignore the subscripts: \\( x_a = x_b
\\). The fact that we bind \\(x_{a}\\) in the abstraction means that
\\(x_{b}\\) must have whatever value we assign to \\(x_{a}\\). \\(x\\) is what
is called a bound variable, because we bound its value by declaring \\(x\\) in
the abstraction.

We can also bind multiple variables to an abstraction, much like how functions
can have multiple inputs:

$$
\lambda x y . x + y \newline
f(x, y) = x + y
$$

You might be confused, because this seems like basic math stuff, obviously the
variables in the body of the function have to correspond to whatever
declarations we have at the beginning of the function. I brought this up for a
reason, however, and that's because lambda calculus also has the notion of
*free variables*, which are variables in a function that don't correspond to
any of the variables that are bound in the head of the function.

As an example, let's define an abstraction with a free variable:

$$
\lambda x . a
$$

In this instance, \\(a\\) can be anything, because it is not bound in the
abstraction. To be clear, \\(x\\) is still bound, it's just not used in the
function (the value assigned to \\(x\\) is effectively thrown away).

### Combinators

Combinators may sound scary, but if you can identify free and bound variables,
you can identify a combinator. A combinator is an abstraction that only has
bound variables.

For example, this is a combinator because all of the variables in the body are
bound:

$$
\lambda x . x + 1
$$

This is not a combinator because it has a free variable:

$$
\lambda x . x + z
$$

### Applications

You can apply variables to abstractions with an *application*. As I've
mentioned before, a function like \\(\lambda x . x\\) allows \\(x\\) to act as
a stand in for any potential value. We can apply actual values to the function,
which I will demonstrate with the identity function:

$$
(\lambda x . x) 1
$$

This statement is saying that we want to apply \\(1\\) in place of \\(x\\) in
the abstraction. Keep in mind, \\(x\\) can be anything. We don't have to apply a
concrete value to it, we can apply another variable:
$$
(\lambda x . x) y
$$

*Note that \\(y\\) is a free variable.*

We can even apply abstractions to other abstractions:

$$
(\lambda x . x) (\lambda y . y)
$$

### Currying

We have seen that abstractions can take multiple arguments. *Currying*, named
after Haskell Curry, allows you to translate a function with multiple arguments
into multiple functions that take one argument. The functions are
mathematically the same, and will evaluate identically.

Let's take a look at a function with multiple arguments:

$$
f(x, y) = x + y \newline
$$

We can translate this to a curried function by splitting up the arguments and
nesting the functions for each argument:

{{< katex >}}
\begin{aligned}
f_{1}(y) &= x + y \\
f_{0}(x) &= f_{1}
\end{aligned}
{{< /katex >}}

The first function, \\(f_0\\), defines x, and returns another function,
\\(f_{1}\\). You then need to pass \\(y\\) to the second function, which will
evaluate to \\(x + y\\).

{{< katex >}}
\begin{aligned}
f_{0}(1) &= f_{1} \\
f_{1}(y) &= x + 1
\end{aligned}
{{< /katex >}}

So we can get the same behavior as \\(f(x, y)\\) by calling \\(f_{0}(x)(y)\\).

With abstractions, we would have something like this:

$$
\lambda x y . x + y
$$

We can convert the abstraction to its curried form like this:

$$
\lambda x . (\lambda y . (x + y))
$$

I added the parenthesis for more clarity, but usually you'd just write it as
\\(\lambda x . \lambda y . x + y\\).

Currying allows you to take contexts where you can only have a single argument
for a function and turn them into functions that allow for multiple arguments.
Currying is not the same as partial application, but they go hand in hand, and
it should be clear why: a partially applied function works because you can
apply arguments one at a time, which will keep returning functions until the
final argument has been applied.

## Reduction

Lambda calculus consists of taking lambda expressions and reducing them using
two operations: alpha equivalence and beta reduction. You take a lambda
expression and you keep reducing it until it can't be reduced any more. If you
can perform these reduction operations, you can do lambda calculus.

We also call the reduced form the *beta normalized* form (since we're reducing
using beta reduction). I'm going to refer to this as just the normalized
form, but there are other normalized forms besides beta-normalized.

### Alpha Equivalence

Alpha equivalence states that the names of variables are not semantically
significant. Remember that variables are just placeholders for potential
inputs. Whether you have a variable named \\(x\\) or \\(z\\) doesn't matter, because
they both allow for any value to be substituted in where they're bound.

Alpha equivalence means that \\(\lambda x . x\\) and \\(\lambda a . a\\) are
equivalent. \\(\lambda x y . x + y\\) and \\(\lambda a b . a + b\\) are also
alpha equivalent.

When you're reducing a lambda expression, you can use alpha equivalence to
rename variables for clarity. For example, if you had a lambda expression
\\((\lambda x y . x + y) (\lambda x . x * x)\\), you might find it helpful to
turn it into \\((\lambda x y . x + y) (\lambda a . a * a)\\) to avoid variable
name collisions, and to make it easier to reason about in your head. The
important thing is that you recognize that the variable names don't matter and
they're just a convenient placeholder for other inputs.

### Beta Reduction

Beta reduction is the substitution step. When you have an abstraction with an
argument applied to it, you need to replace the bound variable in the
body of the abstraction with the argument that's applied to the abstraction.
You have already done this by applying arguments to functions, whether in a
math or programming context, and it's not very different here:

{{< katex >}}
(\lambda x . x + 5) 1 \\
[ x := 5 ]  \\
1 + 5 \\
6 \\
{{< /katex >}}

Note that there is a particular notation for the substitution step:

{{< katex >}}
[x := 1]
{{< /katex >}}

This statement says that all instances of the bound variable \\(x\\) are being
replaced with \\(1\\). This syntax is also what Golang uses to declare new
variables:

```go
x := 1
```

### Convergence and Divergence

Lambda expressions are either *converge* or *diverge*. An expression that
converges will reduce, whereas an expression that diverges will never reduce,
and the reduction process will go on forever.

There is a well-known lambda expression that diverges, called the Omega
combinator:

{{< katex >}}
\Omega = (\lambda x . x x) (\lambda x . x x) \\
[x := \lambda x . x x] \\
\Omega = (\lambda x . x x) (\lambda x . x x)
{{< /katex >}}

This will go on forever.

Here's a simple lambda expression that converges:

{{< katex >}}
(\lambda x . x + 1) 1 \\
[x := 1] \\
1 + 1 \\
2
{{< /katex >}}

All of the lambda expressions I've presented in this article converge (besides
the Omega combinator).

## Further Reading

At this point, you should have a basic understanding of lambda calculus. As you
can see, it's not that complicated! The rules are simple, and it is highly
expressive, and you can do it by hand. While it seems simple on the surface,
lambda calculus has given rise to a lot of theory, and the things that you can
do with it are quite complex.

If you want to go more in depth, I highly recommend you check out these links:

* [Introduction to Lambda Calculus (Chalmers)](http://www.cse.chalmers.se/research/group/logic/TypesSS05/Extra/geuvers.pdf)
* [A Tutorial Introduction to Lambda Calculus (FU Berline)](https://www.inf.fu-berlin.de/lehre/WS03/alpi/lambda.pdf)
* [The Lambda Calculus (Cornell)](https://www.cs.cornell.edu/courses/cs3110/2008fa/recitations/rec26.html)
