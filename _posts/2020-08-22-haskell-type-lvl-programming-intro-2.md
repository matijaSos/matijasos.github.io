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

## Example: Length indexed lists (vectors)

*NOTE: in other tutorials often is used the term "Vector" instead of "List",
which we used so far.*

What is the next piece of information about the list we can know, besides is it empty or not? It is
its length - and if we could knew that, we could implement `safeTail` without problems. We wouldn't
worry what "type" of non-empty is the input list, we just reduce the length for the value of one.

Let's see how we can encode length into the type of our list.

### Encoding list's length in its type

Our list type is going to look like this:

{% highlight haskell %}
data List length a
{% endhighlight %}

Again we have two type parameters, `a` is for type of elements in the list while `length` is for
type from which we can tell the list's length. We also put type params in the different order than
in the previous example in Part 1 because it does not matter.

Before we had just two values (or "states") that we needed to encode with types, `Empty` and
`NonEmpty`, so we created two types (or one type with two data constructors when we used
`DataKinds`). But this time we have infinite amount of values, since length of a list can be
any number. So how do we represent that with types? We can do it in the same way we defined the
list itself, recursively:

{% highlight haskell %}
-- Uninhabitated types
data Zero
data Succ a

type One = Succ Zero
type Two = Succ One
type Three = Succ Two -- or Succ (Succ (Succ Zero))
{% endhighlight %}

`Succ` stands for *Successor*. We can see that since `Succ` is a parametrized type constructor
we can actually from it create infinite amount of types and that solves our problem. This works,
but as in a previous example it would be better to use `DataKinds` so we have type safety
(otherwise somebody can write nonsense such as `Succ Bool` and it would be a valid term):

{% highlight haskell %}
{-# LANGUAGE DataKinds #-}

data Nat = Zero | Succ Nat
{% endhighlight %}

And this is it! From this, `DataKinds` will create type constructors `'Zero` and `'Succ`. Let's
see their kinds and compare them to types of data constructors (`Zero` and `Succ`).

{% highlight haskell %}
-- Types of data constructors
> :type Zero
Zero :: Nat

> :type Succ
Succ :: Nat -> Nat

-- Kinds of type constructors
> :kind 'Zero
'Zero :: Nat

> :kind 'Succ
'Succ :: Nat -> Nat
{% endhighlight %}

The types of data constructors look exactly the same as kinds of newly created type constructors!
This is what `DataKinds` does, promotes data into types, and types into kinds. We ended up with
a new kind `Nat` and we can create infite types with that kind:

{% highlight haskell %}
{-# LANGUAGE DataKinds #-}

data Nat = Zero | Succ Nat

type One = 'Succ 'Zero
type Two = 'Succ One
Type Three = 'Succ ('Succ ('Succ 'Zero))
{% endhighlight %}

Now let's use this and create a type for our length-indexed list:

{% highlight haskell %}
{-# LANGUAGE GADTs #-}
{-# LANGUAGE DataKinds #-}
{-# LANGUAGE KindSignatures #-}

data Nat = Zero | Succ Nat

data List (length :: Nat) (a :: * ) where
    End :: List 'Zero a
    Cons :: a -> Vector length a -> Vector ('Succ length) a
{% endhighlight %}

The main difference from the previous example is how we are managing types in `Cons`. Instead
of just putting e.g. `NonEmpty`, here we are building on top of the input type, length of the input
list. So if the input list had length `'Succ 'Zero`, the new one will have `'Succ ('Succ 'Zero)`.
