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

## [Chapter 10. Parsing a binary data format](http://book.realworldhaskell.org/read/code-case-study-parsing-a-binary-data-format.html)

Besides just parsing, this chapter puts even more focus on how to organize code and use higher-order
functions to reduce boilerplate and duplication (introduces techniques that monads and functors
provide for us).

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

## [Chapter 12. Barcode recognition](http://book.realworldhaskell.org/read/barcode-recognition.html)

This chapter takes as an example a problem of decoding a barcode from the image, but its main
purpose is to introduce new data structures, namely `Data.Array` and `Data.Map`.

Going through the example was a bit lengthy and not so easy to follow in the first reading, but
it seems to not be using a lot of new techniques besides using the mentioned data structures.

To summarize:

* Introduces `Data.Array` and `Data.Map`, demonstrates how to use them on the example of 
recognizing a barcode from the image.

## [Chapter 13. Data Structures](http://book.realworldhaskell.org/read/data-structures.html)

* Associative lists `[(key, value)]` vs Map
* `xs ++ ys` has quadratic complexity, introduces difference list `(5++) . (3++)...`
* `Data.Sequence` offers improved performance for lists

## [Chapter 14. Monad](http://book.realworldhaskell.org/read/monads.html)

Introduces monads formally, explains them on a few exammples:

* `Maybe` monad - helps us avoid "staircasing", unwraping and matching Maybe values continuously.
* `State` monad - helps us pass around a state (e.g. parsing file, generating random values) without
having to do it explicitly.
* Introduced `liftM` method(s) - applies function to the inside of Monad
* Implemented our own Monad - Logger which can log things along doing computations

## [Chapter 15. Programming with monads](http://book.realworldhaskell.org/read/programming-with-monads.html)

* Introduces generalised lifting via `ap` (`<*>`), instead of `liftMN`
* Introduced `MonadPlus` typeclass - useful to avoid `case`
* When using `newtype`, we can enable directive `GeneralizedNewtypeDeriving` - to e.g. derive 
`Monad` typeclass
* Introduced `first` and `second` method for applying functions to pair members
* Introduced multi-param type classes - uh
* "Hiding IO", making it safe - wrap it in a `newtype`
* Introduced `liftIO` - escape hatch from a monad to another monad
* Define monad's interface through a type class - separate interface from an implementation
* Introduced `Reader` and `Writer` monad
