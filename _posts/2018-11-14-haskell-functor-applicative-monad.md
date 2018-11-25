---
layout: post
title: "A little Haskell cheat sheet - Functor vs Applicative vs Monad"
---

When learning about Haskell (from the really good book
["Learn You a Haskell for Great Good"](http://learnyouahaskell.com/)
by Miran Lipovača) I came across the four typeclasses from the title - `Functor`,
`Applicative` and `Monad`.  
They all seem to be very cool and powerful, but after reading through
all of them it was a bit hard to wrap my mind around it - what is exactly the difference between
them, when should I prefer one over another?

This is why I wrote this small cheat sheet - mostly for myself to better remember and understand it,
but if it helps somebody else as well even better!

So let's dig in:

## Functor

### Motivation
If we have `map` for lists and its obvious how useful it is,
why wouldn't we have the same thing for any other type of data structure, like e.g. tree?
So, what `map` is for lists, `fmap` is for any data structure.

More formally, from Data.Functor Hackage
[doc page](http://hackage.haskell.org/package/base-4.12.0.0/docs/Data-Functor.html):

> Functors: uniform action over a parametrized type, generalizing the `map` function on lists.

### One-line explanation
Functor is for mapping over things with a context, such as `[]`, `Maybe`, trees, ...

### Typeclass definition
{% highlight haskell %}
class Functor f where
    fmap :: (a -> b) -> f a -> f b
    (<$>) :: (a -> b) -> f a -> f b
{% endhighlight %}

`Functor` typeclass consists of one function only, `fmap`. `fmap` works like this:

* **input 1**: `(a -> b)` - function that does the mapping (transformation)
* **input 2**: `f a` - value with a context ("wrapped" value)
* **output**: `f b` - transformed value with the preserved context

`<$>` is just the infix version of `fmap`.

### Cool stuff it can do
Mapping over anything, either a predefined data structure or the one you created! How cool is that?

### When to use it
When we have a value (e.g. `Just 10`) or data 
structure (e.g. tree) that provides context along with the data, and we want to transform that
data while preserving the context.

### When not to use it
With functor, we can not "get out" of the given context or change it - e.g. after applying
transformation a list will still be a list, `Maybe`, will still be a `Maybe` etc.

### Examples
In the process of finding cool examples.

* Show mapping over a function (e.g. parametrized newtype, like Parser in Real World Haskell).

### Sources and additional reading
* [LYAH Functor explanation](http://learnyouahaskell.com/making-our-own-types-and-typeclasses#
the-functor-typeclass)
* [Functor design pattern](http://www.haskellforall.com/2012/09
/the-functor-design-pattern.html) - seems really cool and in-depth but have not read it yet


## Applicative

* Map function in a context to the value in a context
* Can be chained together
* We can apply "normal" function to the values with context

## Monad

Most cool stuff.
