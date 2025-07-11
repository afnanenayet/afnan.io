---
title: "Introducing oars"
description: "A Rust library for handling orthogonal arrays"
date: 2019-06-14T21:47:21-04:00
tags: ["orthogonal", "array", "rust", "library", "monte", "carlo"]
math: true
---

## The Motivation Behind `oars`

For a while now, I have been working on a library called
[oars](https://github.com/afnanenayet/oars) (**o**rthogonal **a**rrays
**r**u**s**t).

The need for this library was born out of the work we were doing on my
Bachelor's [thesis](/documents/undergrad_thesis.pdf) and related EGSR
[paper](https://cs.dartmouth.edu/~wjarosz/publications/jarosz19orthogonal.html).
The team and I were working on implementing orthogonal array construction
methods, and using Art Owen's techniques to create point sets from these
orthogonal arrays that are suitable for Monte Carlo integration. At first, we
implemented these algorithms and generation methods and relied on code review
and visualizations to ensure that these points were correct (i.e. valid
orthogonal arrays). As I attempted to implement more complex methods, I found
that I was having a more difficult time catching small errors, and that I was
having a tough time verifying my work in general.  I decided to go ahead and
create something new that would hopefully help me with my work and allow for
faster iteration.

## Orthogonal Arrays

You may be wondering what an orthogonal array (OA) actually is. For an in-depth
read, check out the paper or my [thesis](/documents/undergrad_thesis.pdf). In
this post, I'll give a broader overview.

### Experiment Design

Suppose you're a scientist that wants to measure the effect of various
*factors* on plant growth. We'll say that we are testing out the effects of
water, sunlight, and fertilizer. For each of these factors, we'll say that
there are three *levels* for each factor. Each level can be something like
none, some, and a lot. So for water, we will test out no water, some water, and
a lot of water. Extend this to the other factors.

Now that we have these factors and levels set up, how do we design an
experiment so that we can accurately capture the effects of each combination of
levels and factors? We could simply try out every single combination of levels
and factors, but this seems inefficient.

The idea behind orthogonal arrays is that they provide a better way to stage
experiments, rather than needing us to try every single combination of factors
for each level, we can try out different configurations that require less
trials.

First, let's look at an orthogonal array and see how it relates to experiments.

{.table .table-striped}
sunlight | water | fertilizer
---------|-------|-----------
 0 | 0 | 0
 0 | 1 | 1
 0 | 2 | 2
 1 | 0 | 1
 1 | 1 | 2
 1 | 2 | 0
 2 | 0 | 2
 2 | 1 | 0
 2 | 2 | 1

We can interpret this table as the configuration for each trial of the
experiment. For example, the first trial (the first row of the column),
dictates that we use no sunlight, water, or fertilizer. The second trial is the
same except we use some fertilizer, and on and on. Now you may be wondering,
how do orthogonal arrays get around the issue of having to use every possible
combination of levels for each factor?

There is a property of orthogonal arrays called *strength*. The strength of an
orthogonal array is an integer that must be less than the number of factors.
For an orthogonal array to have a strength $t$ means that if you select any $t$
factors in the orthogonal array, you are guaranteed to find every $t$-tuple of
the levels.

Let's break it down even further. Let's suppose $t=2$. For our levels, which we
can define as the set $\{0, 1, 2\}$, every 2-tuple consists of the following:

* $(0, 0)$
* $(0, 1)$
* $(0, 2)$
* $(1, 0)$
* $(1, 1)$
* $(1, 2)$
* $(2, 0)$
* $(2, 1)$
* $(2, 2)$

This is the same as taking every possible combination of two numbers from the
set we defined above. Now we can take any 2 columns from the orthogonal array,
and those two columns will form this complete list. Of course, we can extend
this all the way up however many factors there are. So for this case, we can
have $t = \{1, 2, 3\}$.

I mentioned earlier that the purpose of orthogonal arrays is to create
*efficient* experiment designs, and we don't want to use every possible
combination (of which there are $s^t$ where $s$ is the number of levels and
$t$ is the number of factors), since that leads to many trials, probably more
than you need. You might realize that when the strength is equal to the number
of factors, we have exactly that case. In the literature, this is a special
degenerate case of an orthogonal array referred as a *full factorial design*.

### Construction Techniques

So you've got these cool arrays telling you how to conduct scientific
experiments, now what? Well for one, you may want to know how you can generate
these arrays. In the literature, many papers are more concerned about the
existence of orthogonal arrays rather than demonstrating how to construct them.
Luckily for us, there are two very efficient construction techniques that were
introduced by Bose and Bush, called the Bose and Bush construction techniques,
respectively.

Oars provides extremely efficient implementations of these construction
techniques. They are in-place (meaning that there is very little
computation/memory overhead), and random access, so each dimension/index of an
orthogonal array can be generated independently and on-the-fly.

### Monte Carlo Sampling

I am going to assume you are familiar with Monte Carlo integration. If you are
not, there are a lot of resources out on the internet that explain this
technique in finer detail.

Suppose you want to integrate a function, $f(x)$. It's a really hard function
to integrate, so you can generate a bunch of random numbers for $x$, evaluate
$f(x)$ for each random input, and then take the average of those numbers to get
an approximation of the integrand. We can pack this into a nice equation:

$$
\int_0^1 f(x) = \frac{1}{N} \sum_{i = 0}^N f(\bar{x})
$$

It looks complicated, but the idea is very simple and intuitive. Of course, you
may be wondering, what do orthogonal arrays have to do with Monte Carlo
integration? Well, it turns out the way that we generate these random numbers,
or our sampling techniques, have a significant effect on how quickly this
technique converges to the right answer.

If you average more random samples, you'll get a more accurate answer. Using
more efficient sample techniques means that you start approaching the correct
answer even faster. Another way to put this is that if we take 100 samples with
pure random sampling, and 100 samples with a more efficient method, the more
efficient method will be more accurate. There are several properties for point
sets that are conducive to efficient Monte Carlo integration, such as
low-discrepancy and good stratification, which I won't get into in this post.

It turns out that if you take an orthogonal array, and you normalize and
randomize it, the resulting point set works very well for Monte Carlo
integration. Art Owen, in his book, provides the equation to transform an
orthogonal array into a point set ready for Monte Carlo integration, and we
implement the method in `oars`.

## Development

The API design for `oars` was heavily inspired by the work that my advisor,
Wojciech Jarosz, did with his tool SampleView. I liked the way his API was
designed - it made sense and it was clean. I simplified the generic API for
orthogonal arrays a little bit, since `oars` isn't a general purpose Monte
Carlo sampling library, but specifically geared towards orthogonal arrays.

One of the first reasons I started using Rust in the first place was because
Rust's testing and benchmarking tools are really easy to use, and provide a
standardized and ergonomic way to implement testing and benchmarks. This sort
of easy verification helped me a lot in ensuring that my implementations of
various orthogonal array construction techniques were not only correct, but
performant. The testing harnesses made it really easy to sanity check all of my
implementations, especially during late nights.

### Numeric Code

One of Rust's strong points is its type system and extensive used of generics.
I wanted my library to be as generic as possible, so I didn't tie my APIs to
any particular numeric type. If you have done any heavy numeric work in Rust,
you know how much of a pain this can be. I used the
[num](https://github.com/rust-num/num) crate to help facilitate numeric generic
traits, but even then, I found the process to be lacking in ergonomics.

Consider the following code snippet in C++:

```cpp
<typename T>
T add_one(T x) {
    return x + 1;
}
```

Now compare this to Rust:

```rust
// We need to use the num crate to have generic methods over numeric types
use num::{NumCast, FromPrimitive};

fn add_one<T>(x: T)
where T: NumCast + FromPrimitive + Copy
{
    x + T::from(1).unwrap()
}
```

While this is a smaller example, we can already see that the Rust code snippet
is much more verbose. My codebase is littered with a lot of these calls, which
can be slightly annoying. I understand why Rust has these safeguards in place,
as there are many small overflow and loss of precision errors that can happen
when C++ engages in reckless implicit casting. I've been a victim of these
errors myself, even in this project. All the same, I wonder if there is a more
ergonomic way we could handle numeric types in Rust and reduce some of the
boilerplate. There is a good reason for this explicitness. Consider the
following code snippet in C++, in which I take a vector of `float`s and try to
sum over the vector.

```cpp
#include <iostream>
#include <numeric>
#include <vector>

float sum_vec(const std::vector<float> &v) {
    return std::accumulate(v.begin(), v.end(), 0);
}

int main() {
    std::vector<float> v{0.2, 0.2, 0.2};
    std::cout << sum_vec(v) << std::endl;
}
```

For those of you who aren't familiar with the `accumulate` method, it's
essentially a reduction over an iterable. You can use different accumulation
methods, such as taking the sum of an iterable or multiplying every element in the
iterable. The function call takes the beginning and end of an iterable,
followed up by the initial value. In this case, I used `0` as my initial value
because I just want the sum of the vector. So what output would you expect to
see from this code snippet? `0.6` right? Wrong! Let's check the output:

```
0
```

We have fallen victim to implicit type coercion. Let's take a look at the
actual signature of `accumulate`:

```cpp
template<class InputIt, class T>
T accumulate(InputIt first, InputIt last, T init);
```

You can see that the return type is the same as the initialization type. If you
look at our function above, we made a subtle mistake and passed in `0` as the
initial value, which is an `int`, rather than `0.0`, which is a `double`. This
led to the method taking each value, (`0.2` in this case), turning it into an
integer, which coerces to `0`, and adding that repeatedly to the initial value
of `0`, which is why the final output was 0.

Here's the corrected version of the function:

```cpp
float sum_vec(const std::vector<float> &v) {
    return std::accumulate(v.begin(), v.end(), 0.0);
}
```

It's a very small detail, but it's an error I fell victim to and spent far too
long debugging because it's so subtle.

The Rust analog for the above code is a little different, since Rust provides
methods on iterators and not methods that take iterators like the STL, so the
analogy to the code snippet would be something like:

```rust
fn main() {
    let v = vec![0.2, 0.2, 0.2];
    let sum: f32 = v.into_iter().sum();
    println!("{}", sum);
}
```

This gives us the expected output: `0.6`.

Now lets see how Rust handles implicit coercions. Suppose that I specified that
my vector was a vector of integers:

```rust
fn main() {
    let v: Vec<i32> = vec![0.2, 0.2, 0.2];
    let sum: f32 = v.into_iter().sum();
    println!("{}", sum);
}
```

`rustc` gives us the following output:

```txt
error[E0308]: mismatched types
 --> src/main.rs:2:28
  |
2 |     let v: Vec<i32> = vec![0.2, 0.2, 0.2];
  |                            ^^^ expected i32, found floating-point number
  |
  = note: expected type `i32`
             found type `{float}`
```

Or we can specify that the return type should be an integer:

```rust
fn main() {
    let v = vec![0.2, 0.2, 0.2];
    let sum: i32 = v.into_iter().sum();
    println!("{}", sum);
}
```

This gives us the following error:

```txt
error[E0277]: the trait bound `i32: std::iter::Sum<{float}>` is not satisfied
 --> src/main.rs:3:34
  |
3 |     let sum: i32 = v.into_iter().sum();
  |                                  ^^^ the trait `std::iter::Sum<{float}>` is not implemented for `i32`
  |
  = help: the following implementations were found:
            <i32 as std::iter::Sum<&'a i32>>
            <i32 as std::iter::Sum>

error: aborting due to previous error
```

Rust's mandated explicitness catches type mismatches that would have been
implicitly coerced in C++, at the expense of some verbosity (at least with
generic numerical types).

### External Libraries

One of the biggest pain points I have with C++ development is library
management. I prefer to use system libraries whenever I can, since I'm on Arch
and I can easily install pretty much every library I'll ever need, but a lot of
times when I'm collaborating with people on different platforms, we have to use
other solutions, like bundling dependencies in tree, or use something like
[Conan](https://conan.io) or [vcpkg](https://github.com/microsoft/vcpkg). While
both managers are really nice, I have issues with both of them, and not
everyone always has them available. More often than not, we simply bundle
dependencies as git submodules.

Rust, on the other hand, has Cargo, which is an excellent package manager. I
really enjoy how easy it is to add a dependency and link it with your project.
Cargo feels much saner than `cmake`, and I don't have to deal with obscure
linker errors.

## The API

As I mentioned before, the API for this project was heavily inspired by the
SampleView program (a program written by Dr. Jarosz to visualize sampling
points in various dimensions for Monte Carlo integration). For each
construction method, I have a struct which holds parameters that are filled in
by the user. There is a trait, `OAConstructor`, which demarcates structs that
can generate orthogonal arrays. This trait contains only one function: `gen()`,
which returns a valid orthogonal array.  Most of the structs that implement
this trait will also likely want some sort of verification method that ensures
that the parameters that users pass in are valid. I didn't create a
standardized interface for this because I wanted to leave some flexibility for
the future and for consumers that implement the `OAConstructor` trait.
Additionally, our research on orthogonal arrays is still fairly early, so I
don't actually know what kind of constraints will come up as we discover new
methods for generating orthogonal arrays.

### Parallelization

I also wanted to create a parallel constructor interface, since it seemed
fairly trivial to convert the Bose and Bush construction methods to utilize
parallelization. `ndarray` provides the `ndarray-parallel` crate, which
integrates with `rayon`. I used rayon's parallel iterators to set up
parallelization with the Bose and Bush methods. These construction methods are
wrapped in a trait called `ParOAConstructor`, which provides a generic
interface for any parallel construction method and can be extended by the user.

Rayon is super easy to use because it can adapt iterators, so if you're writing
function code, the changes to make your routines parallel are typically
one-liners. Since I had my benchmarking tools set up, I used them to see
exactly how much of a benefit parallelization yielded. To my surprise, there
was actually no benefit, since I had tried to parallelize too much. I
experimented a bit, and found that I got the biggest speedup from only
constructing various columns (or dimensions) of an orthogonal array in
parallel. Otherwise, the overhead of threading outweighed the benefits of
staging the tasks in parallel.

### Stateful Types

The implemented constructors in `oars` have parameters that are passed in as
structs. The parameters for Bose and Bush are very specific, your base must be
a prime number, the dimensionality of the resulting point set has to be less
than the base, and so on. At first, I had the structs implement a
function, `verify(&self) -> bool`, that would be called whenever the end user
tried to generate an orthogonal array. I wasn't a huge fan of this implicit
behavior, because verifying large prime numbers can be expensive, and on top of
that, I wanted a more transparent and optional way to do parameter checks.

I learned about stateful types in Rust, and decided to use this principle for
parameter checking as well. The constructors for OAs have two variants: a
checked variant and a "direct" variant. The checked variant is simply a struct
that checks parameters, and upon success, consumes the struct and returns the
regular constructor, which implements the trait for generating OAs.

The general flow for the verification states is as follows:

```rust
impl<T> BoseChecked<T> {
    verify(self) -> OarsResult<Bose> {
        // verify things here
        // ...

        // invalid parameters
        return OarsError::new("bad params");

        // valid parameters
        Bose {
            // ...
        }
    }
}
```

What happens here is that the user supplies some values to the struct. This
method performs some checking, and upon success, the `BoseChecked` struct is
consumed so it can't be used anymore, and is replaced by the `Bose` struct,
which implements the proper traits so it can generate an orthogonal array. Of
course, if you don't want to incur the overhead of checking the parameters, you
are free to just use the `Bose` struct directly, but improper parameters will
lead to invalid orthogonal arrays, so it is up to the user to verify their own
parameters.

The stateful typing system allows for some pretty ergonomic code. The general
flow for generating orthogonal arrays with checked code goes like this:

```rust
let bose = BoseChecked {
    prime_base: 3,
    dimensions: 3,
};

let oa = bose.verify()?.gen()?;
```

### Procedural Macros

One of the consequences of utilizing stateful types for my library was that
each constructor struct needed to have a variant that had the exact same
fields. So for example, I have a struct for the `Bose` constructor, as well as
a struct called `BoseChecked`. The library follows a consistent naming
convention where the checked variant of a constructor (`Ctor`) is suffixed by
"Checked", so `Ctor -> CtorChecked`. As you can imagine, it would be really
annoying to have to create duplicate structs for each constructor, and on top
of that, you introduce the potential for copy/paste errors.

Luckily, Rust has a solution for this in the form of procedural macros. If
you're not familiar with Rust's macros, they are very powerful tools that are
also much safer than the macros you encounter in C/C++. Whereas those macros
are simple text substitution and allow for a lot of error, Rust's macro system
is checked by the compiler, and has some very strong reflection capabilities. I
created a derive macro called `Checked` which automatically creates a copy of a
struct, but changes its name to be suffixed by `Checked`. Usage is very simple:


```rust
// creates a struct called `BoseChecked` with identical parameters, comments,
// visibility, etc.
#[derive(Checked)]
struct Bose {
    // ...
}
```

Developing this macro was very nice because the compiler was able to catch
errors as I developed. Had I used simple text substitution, catching bugs would
be a much more arduous and lengthy process.

The macro itself actually creates a syntax tree from the code block that it's
applied on, so if for example, this macro is applied to something besides a
struct, the macro will invoke a compilation error. Additionally, modifying a
syntax tree rather than working with text means that the way I am replacing the
identifier for the struct is much easier and safer than doing complex string
parsing.

## Resources/Links

* [EGSR paper](https://cs.dartmouth.edu/~wjarosz/publications/jarosz19orthogonal.html)
* [Bachelor's thesis](https://www.cs.dartmouth.edu/~trdata/reports/abstracts/TR2019-872/)
* [Library documentation](https://docs.rs/oars)
* [Github link](https://github.com/afnanenayet/oars)
* [Art Owen's book](https://statweb.stanford.edu/~owen/mc/)

_Thanks to Zach Misso, Lucy Xiao for pointing out improvements/errata._

---

See discussion on [Reddit](https://www.reddit.com/r/rust/comments/c7vz61/introducing_oars/)
