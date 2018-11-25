---
layout: post
title: "Book: Real World Haskell"
---

[Real World Haskell](http://book.realworldhaskell.org/) is a great book with examples on how
to use and apply Haskell to the real world problems.

While reading it I put here a breakdown of important points I learn in each chapter, so I can
better memorize and return later to it when I need it.

## [Chapter 9. I/O case study - a library for searching the filesystem](http://book.realworldhaskell.org/read/io-case-study-a-library-for-searching-the-filesystem.html)

* Introduces combinators - function that takes other functions as arguments and returns new
function

## [Chapter 10: Parsing a binary data format](http://book.realworldhaskell.org/read/code-case-study-parsing-a-binary-data-format.html)

* Prefer ADS (algebraic data structures) over e.g. tuples for passing the state -> ADS is easier
to modify/extend since it does not depend on the internal structure of the data (like tuples do).
* Don't put constraints when defining a data type, but rather on specific functions that require
it. That way we keep our code more general and don't make unneccessary assumptions.
* Introduces a term of lifting a function - make it work with parametrized types.
* Functor - not only for values "inside" of a container, but also for functions.
* Introduces monad techniques for handling context by implementing them themselves.

## [Chapter 11. Testing and quality assurance](http://book.realworldhaskell.org/read/testing-and-quality-assurance.html)

Introduces [QuickCheck](http://hackage.haskell.org/package/QuickCheck) as a testing tool for Haskell. What is cool is that it can generate test cases
for us.

* We can define a property for which we want to always hold true - QuickCheck will randomly generate
test cases for us
* We can define a model implementation and also test that (e.g. our implementation of sort vs.
standard library sort) -> no special syntax, uses the same mechanism as above
* With HPC (Haskell Program Coverage) we can observe test coverage of our code in details and
identify the weak spots

Further reading:

* [The Design and Use of QuickCheck](https://begriffs.com/posts/2017-01-14-design-use-quickcheck.html) - seems 
to be a cool and recent (2017.) tutorial
* [QuickCheck manual](http://www.cse.chalmers.se/~rjmh/QuickCheck/manual.html) - a bit outdated (they state), but still good
