---
layout: post
title: "A little Haskell cheat sheet - Functor vs Applicative vs Monoid vs Monad"
---

When learning about Haskell (from the really good book
["Learn You a Haskell for Great Good"](http://learnyouahaskell.com/)
by Miran Lipovača) I came across the four typeclasses from the title - `Functor`,
`Applicative`, `Monoid` and `Monad`.  
They all seem to be very cool and powerful, but after reading through
all of them it was a bit hard to wrap my mind around it - what is exactly the difference between
them, when should I prefer one over another?

This is why I wrote this small cheat sheet - mostly for myself to better remember and understand it,
but if it helps somebody else as well even better!

So let's dig in:

## Functor

* For mapping over things with a context
* We can map over a list, Maybe, Tree, ...

## Applicative

* Map function in a context to the value in a context
* Can be chained together
* We can apply "normal" function to the values with context

## Monoid

* Type for which we have: binary function + identity
* Useful for folding data structures (fmap)

## Monad

Most cool stuff.
