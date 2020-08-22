---
layout: post
title: "Intro to type-level programming in Haskell - Part 2"
---

*This is the second part of the series of posts on this topic. You can find the first part
[here](http://matija.me/2020/07/04/haskell-type-lvl-programming-intro/).*

In the previous part of this tutorial we created a `List a empty` for which we could tell from its
type whether it is empty or not (by `empty` being other `Empty` or `NonEmpty`). This is how our
final code looked like:

{% highlight haskell %}
{-# LANGUAGE GADTs #-}
{-# LANGUAGE DataKinds #-}
{-# LANGUAGE KindSignatures #-}

data ListStatus = Empty | NonEmpty

data List (a :: * ) (empty :: ListStatus) where
    End :: List a 'Empty 
    Cons :: a -> List a empty -> List a 'NonEmpty

safeHead :: List a NonEmpty -> a
safeHead (Cons elem _ ) = elem
{% endhighlight %}

The question where we left off was: can we create `safeTail` function that would accept only
non-empty lists?

Let's try to construct its type signature:
{% highlight haskell %}
safeTail :: List a NonEmpty -> List a what?
safeTail (Cons elem rest) = rest
{% endhighlight %}

Everything makes sense except one thing - we don't know what is `empty` going to be for a returned
list. It depends on the input list - if it has only one element, the result will be `Empty`.
Otherwise, it will be `NonEmpty` But the problem is that with our current system we cannot 
differentiate between them and they are both marked as `NonEmpty`.

The conclusion is: we need a more expressive type!

## Example: Length indexed vectors (lists)

What is the next piece of information about the list we can know, besides is it empty or not? It is
its length - and if we could knew that, we could implement `safeTail` without problems. We wouldn't
worry what "type" of non-empty is the input list, we just reduce the length for the value of one.

