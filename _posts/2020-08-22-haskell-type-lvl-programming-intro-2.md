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
safeTail :: List a NonEmpty -> List a ?
safeTail (Cons elem rest) = rest
{% endhighlight %}

Everything makes sense except one thing - we don't know in compile time what is `empty` going
to be for a returned list.
It depends on the input list - if it has only one element, the result will be `Empty`.
Otherwise, it will be `NonEmpty` But the problem is that with our current type system we cannot 
differentiate between them and they are both marked as `NonEmpty`.

### [Going deeper] But what if we just put `List a smth` as a return type?

In the previous part of the tutorial we had cases where the created list had `empty` type param
which just stayed general and that worked. So what would happen if we tried that here?

Ok, so here is what we are trying to compile:
{% highlight haskell %}
safeTail :: List a NonEmpty -> List a smth
safeTail (Cons elem rest) = rest
{% endhighlight %}

This looks like we are trying to leave to the compiler to determine what will `smth` actually be,
we just say "this will be something/anything". Plus we have a type param value `smth` that doesn't
appear anywhere in the function arguments, on the "left" side, but just in the result. I am not
sure if that could make sense in any case?

But interesting thing is, the following compiles:
{% highlight haskell %}
safeTail :: List a NonEmpty -> List a smth
safeTail (Cons elem rest) = undefined
{% endhighlight %}

It doesn't say *"you have type param value `smth` which doesn't appear anywhere on the left side."*
which surprised me a bit. I still wonder why could that be valid?

But if we try to compile the original function, it fails. The error message is a bit longer as
usual with Haskell, but from what I understood it basically says the following: *"I can see from
the `List` constructors that any list you create will have a specific type, and you are here
trying to say it will be just anything. Tell me exactly what it is."*

*TODO*: I am not sure if I got this completely right. Figure out what is exactly the reason this
does not compile.

The conclusion from all this is: we need a more expressive type so we can specifically say what
is the type of the returned list going to be!

## Example: Length indexed lists (vectors)

*NOTE: in other tutorials often is used the term "Vector" instead of "List",
which we used so far.*

What is the next piece of information about the list we can know, besides is it empty or not? It is
its length - and if we could knew that, we could implement `safeTail` without problems. We wouldn't
worry what "type" of *non-emptiness* input list has -  we would just reduce the length by one.

Let's see how we can encode length into the type of our list.

### Encoding list's length in its type

Our list type is going to look like this:

{% highlight haskell %}
data List length a
{% endhighlight %}

Again we have two type parameters, `a` is for type of elements in the list while `length` is for
type from which we can tell the list's length.

Before we had just two values (or "list states") that we needed to encode with types, `Empty` and
`NonEmpty`, so we created two types (or one type with two data constructors when we used
`DataKinds`). But this time we have infinite amount of values, since length of a list can be
any natural number. So how do we represent that with types?
We can do it in the same way we defined the list itself, recursively:

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

And this is it! From this, `DataKinds` will create type constructors `'Zero` and `'Succ nat`. Let's
see their kinds and compare them to types of the data constructors they originated from
(`Zero` and `Succ`).

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
This is what `DataKinds` does - promotes data into types, and types into kinds. We ended up with
a new kind `Nat` and we can create infite amount of types with that kind:

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
    Cons :: a -> List length a -> List ('Succ length) a
{% endhighlight %}

The main difference from the previous example is how we are managing types in `Cons`. Instead
of just putting e.g. `NonEmpty`, here we are building on top of the input type, length of the input
list. So if the input list had length `'Succ 'Zero`, the new one will have `'Succ ('Succ 'Zero)`.

And that is it! We have now defined list which will have its length contained in its type. Let's
see it in action:

{% highlight haskell %}
> :t End
End :: List 'Zero a

> oneElemList = Cons "elem" End
> :t oneElemList
oneElemList :: List ('Succ 'Zero) [Char]

> twoElemList = Cons "elem2" oneElemList
> :t twoElemList
twoElemList = List ('Succ ('Succ 'Zero)) [Char]
{% endhighlight %}

Wohoo, it works! Now we can finally implement `safeTail`:

{% highlight haskell %}
safeTail :: List ('Succ n) a -> List n a
safeTail (Cons _ rest) = rest
{% endhighlight %}

Let's test it out:

{% highlight haskell %}
> emptyList = End
> safeTail emptyList
Error: Couldn't match type 'Zero with 'Succ n

> oneElemList = Cons "elem" End
> safeTail oneElemList
End
{% endhighlight %}

Everything works as expected. This time we didn't have any trouble defining `safeTail`'s types -
we restricted input list to have at least one element (`'Succ n`) and then output will simply be
of length `n`.

The basic implementation of length-index list is now done. We can create a list and we can also
operate with `safeTail` and `safeHead` on it. Is there anything else we might need?

### List concatenation - making it work on type level

For example, what if we wanted to concatenate two lists? Logically, we know that the resulting list
will have length which is sum of lengths of the input lists. But how do we represent it on type
level?

Let's try to see what will the function signature look like. We will call this function `append`
(so we don't confuse it with Prelude's `concat`) - it will take two lists and produce a new list
which is a concatenation of those two input lists:

{% highlight haskell %}
append :: List n a -> List m a -> List ? a
{% endhighlight %}

Let's also see a few examples of how it would work:

{% highlight haskell %}
> twoElems = append (List "a" End) (List "b" End)
> :t twoElems
twoElems :: List ('Succ ('Succ 'Zero)) [Char]

> twoEmptyLists = append End End
> :t twoEmptyLists
twoEmptyLists :: List 'Zero a
{% endhighlight %}

So we take two lists of lengths `n` and `m` respectively (we don't care if
there are empty or not in this case, since there isn't any "unsafe" scenario) and we
produce a new list of length `n + m`. But how do we represent its
length, what do we put in place of `?`?  
Obviously, we want to sum `m` and `n` - but how do we do that on the type level?

### Type families - functions that operate on types

Turns out there is a special mechanism for this in Haskell and it is provided in form of another
language extension named `TypeFamilies`.
