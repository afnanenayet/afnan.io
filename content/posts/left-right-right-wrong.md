---
title: "Why Can't Left Be Right and Right Be Wrong?"
date: 2020-08-16T23:41:59-04:00
draft: true
---

## The `Either` Type

When I first learned about error handling in Haskell, specifically the `Either`
type that's used to return a value that can *either* be an error or a "correct"
result. Rust users will know this as the `Result<T, E>` type.

We can declare the `Either` type like this in Haskell:

```haskell
data Either a b = Left a | Right b
```

You can find the full documentation on the type
[here](https://hackage.haskell.org/package/base-4.14.0.0/docs/Data-Either.html).

In Haskell, it is convention to denote an error type with the `Left` data
constructor, and a correct result with the `Right` data constructor. When I
first learned about this, I wondered why exactly we use `Left` for errors and
`Right` for correct results, but I chalked it up to convention. It turns out
that there a more principled reason these associations. The reason involves
kinds, typeclasses, and a specific typeclass called `Functor`.

## Background

### Kinds

In order to understand typeclasses like `Functor`, I think it's important to
first dive into *kinds*. Simply put, a kind is like the type of a type
constructor. They look very similar to the type definition of a function, for
example:

```haskell
Int :: * -- kind
data Int -- type definition

someConstant :: Int -- type
someConstant = 1 -- function definition

Box a :: * -> * -- kind
data Box a = Box a -- type definition

someFn :: a -> a -- type
someFn x = x -- function definition

HigherKindedBox :: (* -> *) -> * -> * -- kind
data HigherKindedBox f a = HigherKindedBox (f a) -- type definition

higherOrderFn :: (a -> a) -> a -- type
higherOrderFn f a = f a -- function definition
```

It's ok if you don't fully understand what's going on with the kinds, but you
should be able to tell that whatever is going on with kinds and types looks
very similar to what's going on with types and functions. It's almost like the
kind is the type...of a type, and that's exactly what it is! A kind is just the
type of a type constructor.

In Haskell, the kind of a basic type constructor that isn't parametrized by
another type (also called a *nullary* type constructor) is `*` (which is
pronounced "star"). A very common *unary* type constructor is the list type.

Let's define a singly linked list:

```haskell
data List a = Nil | Cons a (List a)
```

The kind of `List` is `* -> *` because it needs another type before we can
determine the fully-inhabited type of `List`. For example, we could create a
list of `String`s or a list of `Integer`s, but we don't know what kind of list
it's going to be we supply some type to the compiler. `List Int`, on the other
hand, is an *inhabited type constructor* and has the kind `*`.

Let's go one step further with `Either`. The declaration for `Either` is listed
above, and you can see that it takes two types, so its kind is `* -> * -> *`.
Just like with functions, type constructors are curried, so we can actually
have a partially applied type. In this case, we could define some partial type
`Either MyErrorType`, which has the kind `* -> *`, because it requires `b` to
be fully inhabited.

Just like we can have higher-order functions (HOFs), which are functions that
can take other functions as parameters or return a function, we can have
higher-kinded types (HKTs), which are type constructors that can accept other
type constructors. Things get a little more interesting here. Let's take a look
at our `HigherKindedBox` type. In its data constructor, we notice that it holds
a type `a`, which is applied to `f`, which means that `f` has to be some type
that accepts a type as a parameter, so it must have the kind `* -> *`.

Let's see what happens when we try to supply two integers:

```
Prelude> HigherKindedBox (1 1)
<interactive>:10:1: error:
    • Non type-variable argument in the constraint: Num (t -> f a)
      (Use FlexibleContexts to permit this)
    • When checking the inferred type
        it :: forall t (f :: * -> *) a.
              (Num t, Num (t -> f a)) =>
              HigherKindedBox f a
Prelude>
```
