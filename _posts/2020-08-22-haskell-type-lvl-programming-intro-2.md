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

*NOTE*: On the other hand, we can have `Nothing :: Maybe a` and in that case it is ok to have `a` on
the right side only, so I guess that is ok actually.

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

What is the next piece of information about the list we could know, besides whether it is empty or not? It is
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
`DataKinds` - then from that `DataKinds` created the types for us).
But this time we have infinite amount of values, since length of a list can be
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
but as in a previous example it would be better to use `DataKinds` so we have kind safety
(otherwise somebody could write nonsense such as `Succ Bool` and it would be a valid term):

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
will have length which is the sum of lengths of the input lists. But how do we represent that on type
level?

Let's try to see what will the function signature look like. We will call this function `append`
(so we don't confuse it with Prelude's `concat`) - it will take two lists and produce a new list
which is a concatenation of those two input lists:

{% highlight haskell %}
append :: List n a -> List m a -> List ? a
{% endhighlight %}

Let's also see a few examples of how it would work:

{% highlight haskell %}
> twoElems = append (Cons "a" End) (Cons "b" End)
> :t twoElems
twoElems :: List ('Succ ('Succ 'Zero)) [Char]

> twoEmptyLists = append End End
> :t twoEmptyLists
twoEmptyLists :: List 'Zero a
{% endhighlight %}

So we take two lists of lengths `n` and `m` respectively (we don't care if
they are empty or not in this case, since there isn't any "unsafe" scenario) and we
produce a new list of length `n + m`. But how do we represent its
length, what do we put in place of `?`?  
Obviously, we want to sum `m` and `n` - but how do we do that on a type level?

Wouldn't it be handy if there was a way to define a function that operates on types? Then we could
define things like this:

{% highlight haskell %}
append :: List n a -> List m a -> List (Add n m) a
{% endhighlight %}

Where `Add n m` is a "type" function that would take two types `n` and `m` and produce a new type
which would represent their sum.

### Type families - functions that operate on types

Turns out there is a special mechanism for this in Haskell and it is provided in form of another
language extension named `TypeFamilies`. This is how we would define our type family `Add n m`:

{% highlight haskell %}
type family Add (x :: Nat) (y :: Nat) :: Nat where
Add 'Zero n = n
Add ('Succ n) m = Add n ('Succ m)
{% endhighlight %}

Since we are dealing with recursive types (`'Succ`), this type function has to work in the same way.
In the base case when the left type is `'Zero`, we simply return the second type. In the general
case we keep "deconstructing" the left argument and "piling" it on the right type until we
reach the base case.

This looks like a logical way to do it. But if, we try to compile it we will get the following
error:

{% highlight haskell %}
• The type family application
    is no smaller than the instance head ‘Add n ('Succ m)’
    (Use UndecidableInstances to permit this)
• In the equations for closed type family ‘Add’
    In the type family declaration for ‘Add’
{% endhighlight %}

The problem is that GHC is here scared that our general recursion case might never terminate. 
I checked out docs of `UndecidableInstances` and here is what it says: 
*These restrictions ensure that
instance resolution terminates: each reduction step makes the problem smaller by at
least one constructor.*

And this is exactly what is the problem: we made a reduction step, but we still have
the same number of constructors (`'Succ`) - we just moved it from one argument to another and that
made GHC suspicious we are designing a system that actually terminates.

If we e.g. did this (although it is conceptually wrong, does not produce the result we want):

{% highlight haskell %}
Add ('Succ n) m = Add n  m
{% endhighlight %}

we wouldn't get any errors - GHC sees that we got rid of one `'Succ` and is happy and convinced
that we will eventually come to the end.

*TODO: Why GHC doesn't complain if we do `Add ('Succ n) m = 'Succ (Add n m)`, since we still have the same number
of constructors? Is it because `Add` is not at the beginning anymore?*

This was pretty impressive for me, I didn't know GHC watches over this kind of stuff, "counting" how
our recursion is doing.

Anyhow, we can solve the problem from above by introducing `UndecidableInstances` extension. With it, our code
from above successfully compiles.

Now that we know how to sort out things on the type-level, let's actually write the function which appends two
lists:

{% highlight haskell %}
append :: List n a -> List m a -> List (Add n m) a
append End xs = xs
append (Cons elem rest) xs = Cons elem (append rest xs)
{% endhighlight %}

We again do it recursively, always taking the first element "out" and calling `append` again for the reduced first
list and the second list (general case). When first list reaches `End`, `append` will just return the
second list (base case).

But, if we try compile this we get an error! This is what it says:
{% highlight haskell %}
• Could not deduce: Add n1 ('Succ m) ~ 'Succ (Add n1 m)
      from the context: n ~ 'Succ n1
        bound by a pattern with constructor:
                   Cons :: forall a (n :: Nat).
                            a -> List n a -> List ('Succ n) a,
                 in an equation for ‘append’
        at gadtNonEmptyList.hs:121:9-20
      Expected type: List (Add n m) a
        Actual type: List ('Succ (Add n1 m)) a
    • In the expression: Cons a (append rest xs)
{% endhighlight %}

What this error says is it basically complains that return type is not correct, it is different from what we
declared in the function signature. In the signature we said that return type is `List (Add n m) a`. When
`append`'s function body is evaluated, `Cons elem (append rest xs)`, it will be of
type `List ('Succ (Add n m) a)`.

Why? Let's deconstruct `Cons elem (append rest xs)` from the inside out. `append rest xs` is of
type `List (Add n m) a`. And `Cons`'s type signature is `Cons :: a -> List n a -> List ('Succ n) a` - whatever `n`
is, `Cons` will put `'Succ` on it. So in our case, where `n` is `Add n m` (from `append rest xs`), applying `Cons`
on it will produce type `List ('Succ (Add n m)) a`. And what we promised to GHC in `append`'s return type was
`List (Add n m) a`.

Even if GHC tries to evaluate `Add n m` to see what is under the hood, it will again see from the general step
that `Add x y` is produced.
So when comparing returned and expected type (`List ('Succ (Add n m)) a` and `List (Add n m) a`), GHC 
sees that extra `'Succ` and
concludes *"wait, this is not what I expected, I was looking for
`Add` but I got `'Succ`"* and throws an error.

The root of the problem is in what we experienced when defining `Add n m` type family,
that GHC doesn't understand that `Add n m` will eventually produce a concrete type (`'Succ` in this case),
because of the way we designed our recursion which always puts `Add` in the front.
We managed previously to patch it up with enabling `UndecidableInstances`, but now it is coming back
to haunt us again.

We can solve it by slightly changing `Add`'s definition, the general case of the recursion:
{% highlight haskell %}
type family Add (x :: Nat) (y :: Nat) :: Nat where
Add 'Zero n = n
Add ('Succ n) m = 'Succ (Add n m)
-- Before: Add ('Succ n) m = Add n ('Succ m)
{% endhighlight %}

So what we did is rearrange things a bit so now `'Succ` comes in front. With this change we don't get any errors
and we can also disable `UndecidableInstances`. When GHC sees this it decides our
recursion will eventually terminate and doesn't complatin. I am not sure exactly why, my guess is because here we
applied `Add` to the "reduced" argument (`n`, while input was `'Succ n`) so that tells GHC that our recursion
is moving in the right direction, that we are reducing the problem.
Previously we just moved `'Succ` from one argument to another so I guess that was
the problem, we haven't "reduced" anything. This is my current assumption and I would still like to understand this
better.

And this is it, with this last change everything works as expected! Now we can see `append` in action, but let's
first just do one more thing:
{% highlight haskell %}
instance Show a => Show (List n a) where
    show End = "End"
    show (Cons e es) = show e ++ " - " ++ (show es)
{% endhighlight %}
I implemented `List` as an instance of `Show` typeclass so we can nicely visualize our lists.

Let's now see `append` on a few examples:
{% highlight haskell %}
> append (List "a" End) (List "b" End)
"a" - "b" - End

> :t append ((List "a" End) (List "b" End))
 append ((List "a" End) (List "b" End)) :: List ('Succ ('Succ Zero)) [Char]

> append (List "a" End) End
"a" - End
{% endhighlight %}

We can see how everything works and also that types accurrately reflect number of elements in the list.

For the end, here some of my thoughts on type-level programming after trying it out on these
examples:
* Type-level programming let's us put put more features/properties into types and achieve compile
time safety for them, but not without a cost - we have to program things both on type and
data level (e.g. as we did with `append` function).
* It is important to consider how we will implement and structure types to match well in all instances, since
it is possible run into non-obvious errors such as we hadd with `Add n m`.

All together it was really cool for me to learn about this! I hope you found it useful - let me know if you
have any questions or if I could have done anything better.
