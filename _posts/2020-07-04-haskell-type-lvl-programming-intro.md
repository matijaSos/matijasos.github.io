---
layout: post
title: "Intro to type-level programming in Haskell - Part 1"
---

I heard a lot lately about using types in Haskell to describe function arguments in more details
(e.g. function takes a list that is non-empty) and thus provide higher compile-time safety.
It sounded cool so I decided to research more about it and I created this blog post as a memo of
what I learned so far.

Resources I used:
- [Tutorial on GADTs and DataKinds by andre.tips](https://web.archive.org/web/20200212212525/https://andre.tips/wmh/generalized-algebraic-data-types-and-data-kinds/)
    - explains GADTs well, but DataKinds not so much. But would recommend to start with this one, although link seems to be down currently?
- [Basic Type Level Programming in Haskell by Matt Parsons](https://www.parsonsmatt.org/2017/04/26/basic_type_level_programming_in_haskell.html)
    - pretty extensive, but not so intuitive. Still the most complete intro (and pretty much
the only one) to the topic. Most of my examples below are taken from it.

## Why type-level programming?

As I wrote above, my current understanding is that with it we can add more info into the type
signature of a function, making it "safer" in the compile time.
E.g. instead of just saying *"this function takes a list"*
we can say *"this function takes a non-empty list"*, or
*"this function takes Int which is > 10 and < 127"* (although not yet sure how this last one
would be done).

## Example: A function that accepts only a non-empty list

We want to be able to tell from its type whether a list is empty or not. To do that we will
create a new type which will have that information stored in it.

First, let's see how we would create a "normal" list type on our own:
{% highlight haskell %}
data List a = End | Cons a (List a)
{% endhighlight %}

This is a usual recursive definition of list (`Cons` stands for *Constructor*). Once there is
an instance of this type we can tell which type of elements the list contains (e.g. `List Int`
or `List (Maybe String)`), but nothing more than that. We cannot deduce from its type (which means
in compile time) whether it is empty or not.  

We could of course check it in runtime and throw an exception if the list is empty, but we want to
be stricter than that. We want to ensure that program cannot even be compiled if an empty list is
provided where it shouldn't be.

### What are we missing?

The problem with the "normal" list we defined above is that we are missing information in its type.
We have only one piece of
information and that is type of the elements within the list - we use type parameter `a` to declare
that. We can also use it to define functions that work only for a specific `a`. E.g. here is a
function that works only on a list of integers:

{% highlight haskell %}
sumListElements :: List Int -> Int
{% endhighlight %}

This is guaranteed in compile-time. If we call this function with a list of e.g. `Bool`s, the
compiler will throw an error at us.

### Using type params to encode extra information

So here's an idea - why don't we just use the same mechanism again (having a type parameter) to
know whether a list is empty or not. If we added another type parameter to keep track of that,
our type (disregarding data constructors for now) would look like this:

{% highlight haskell %}
data List a empty
{% endhighlight %}

Just as type param `a` means *any type* (e.g. `Int` or `MyType`), the same applies for `empty`.
E.g. we could have `List Int Double` or `List String Bool` or `List Int SomeCustomType`.
Just as we restricted function `sumListElements` above to work only when `a` is `Int`, we can use
`empty` in the same way.

Let's say we want to implement a safe version of `head` function - that means it accepts only a
non-empty list as an argument, otherwise it won't compile. We will call it `safeHead`.

Ok, we introduced `empty` as another type parameter, but the question we are facing now is what
do we do with it? Which concrete types will take its place and how?

Lets introduce two new types:

{% highlight haskell %}
data Empty
data NonEmpty
{% endhighlight %}

The interesting thing here is that we have only type constructors for these types and no data
constructors. That means we cannot create instances (values) of these types, but that is ok!
We need these types only at the type level, in function signatures. Such types are also called
[*uninhabited* or *empty*](https://wiki.haskell.org/Empty_type).

Now let's imagine we have a way to correctly assign `Empty` and `NonEmpty` to empty and non-empty
lists' types. Then we could define `safeHead` as follows:

{% highlight haskell %}
safeHead :: List a NonEmpty -> a
safeHead (Cons elem _ ) = elem
{% endhighlight %}

We wouldn't even have to define a case for `End` since the type guarantees it can't ever happen.

### Assigning correct type to `empty`

The main question that is left is how do we produce such lists with an extra type parameter, and
how do we make sure which type `empty` takes when? This is what we will look at now.

This is how our type `List` looks once we have added `empty` as a second type parameter:

{% highlight haskell %}
data List a empty = End | Cons a (List a empty)
{% endhighlight %}

Except adding that extra type parameter, nothing else changed. When I first saw this,
I was confused by
the fact there is a type parameter on the left side that doesn't appear anywhere on the right side
as a data.
How is that possible, why would that make sense? (Ok, *there is* `empty` on the right side here, but
only as a part of a type designation and not as a part of data constructor. Which means there will
never be anything of type `empty` in some value of this type).

But turns out it does make sense, since we use it as a designation at the type level only, to show
that an
underlying value has a certain property (empty or non-empty in this case). Such types, which have a
type parameter(s) on the left side that doesn't appear on the right, are
also called [*phantom types*](https://wiki.haskell.org/Phantom_type).

Let's see now what happens if we create an instance of our new `List` and test its type in GHCi:

{% highlight haskell %}
> emptyList = End
> :t emptyList
List a empty

> nonEmptyList = Cons "haskell" End
> :t nonEmptyList
nonEmptyList :: List [Char] empty
{% endhighlight %}

As we can see, GHCi concluded that `a` is a string in `nonEmptyList`, but could not deduce anything
for `empty` in either case, since it is not used anywhere. So how can we solve that
and make sure that `empty` becomes `Empty` for `emptyList` and `NonEmpty` for `nonEmptyList`?

### GADTs

Before we continue, let's check types of our data constructors, `End` and `Cons` (since they are
functions as well, we can do that):

{% highlight haskell %}
> :t End
List a empty

> :t Cons
a -> List a empty -> List a empty
{% endhighlight %}

We can see they have no power to change or specify `empty` type param in any way. `End` will leave
it unspecified, while `Cons` will preserve it from the input list. Also, we have no way to change
these type signatures as they are automatically derived from `List` type definition.

This is exactly where GADTs come in. GADTs ([Generalized Algebraic Data Types](
https://en.wikibooks.org/wiki/Haskell/GADT)) is a Haskell extension that lets us explicitly
define the type signatures of data constructors.

Before seeing GADTs in action, let's first remind ourselves of the standard,
non-GADT definition of `List` we used above:

{% highlight haskell %}
data List a empty = End | Cons a (List a empty)
{% endhighlight %}

Now let's rearrange it a bit and add types of data constructors in comments so we can more easily
reason about them:

{% highlight haskell %}
data List a empty =
    End |                   -- End :: List a empty
    Cons a (List a empty)   -- Cons :: a -> List a empty -> List a empty
{% endhighlight %}

As we mentioned, the types in comments are automatically derived and we cannot control them. But
that is exactly what we want to do, and GADTs let us achieve that using the following syntax:

{% highlight haskell %}
{-# LANGUAGE GADTs #-}

data List a empty where
    End :: List a Empty
    Cons :: a -> List a empty -> List a NonEmpty
{% endhighlight %}

We can see it is very similar to our "rearranged" `List` definition above! What GADTs let us do
is write by ourselves types of data constructors (which were in the previous definition in the
comments), giving us control to specify them as we wish!

The difference in the syntax is that we have to add `where` after the type name and then
for each data constructor we specify its type signature.

Now we finally have the power to control `empty` type parameter (in `List a empty`)! We specified
that `End` will mark list as `Empty`, while `Cons` will mark it as `NonEmpty`. And this is exactly
what we wanted to do, because if we used `End` we know the list is empty, while if we used `Cons`
we know there is at least one element in it, which makes it non-empty.

Let's see it in action! Using it stays the same as without GADTs, just that this time there will
be `empty` type param which assumes an appropriate type:

{% highlight haskell %}
> :t End
End :: List a Empty

> :t (Cons 5 End)
Cons 5 End :: Num a => List a NonEmpty
{% endhighlight %}

Wohoo, this works now! We see we can construct values of this type and we will always know whether
it is empty or not. `safeHead` function we defined above will work on these without any problems.

### Can we just use smart constructors instead of GADTs?

One possible "downside" of GADTs is that it is a language extension we have to enable, thus making
our codebase a bit heavier (longer compilation time?) and less "standard".

Sometimes we can avoid using GADTs with smart constructors. Let's see what that is and how it would
work in this case.

[Smart constructor](https://wiki.haskell.org/Smart_constructors) is simply a function that is used
to create a certain value instead of using its
data constructor directly. We typically do that (hide data constructors and expose smart
constructor functions) when we want to have extra control over the value creation. E.g. we want to
make some extra checks, or make sure an invalid value isn't provided etc.

For example, we could provide a following smart constructor to create an empty list:

{% highlight haskell %}
data Empty
data NonEmpty

data List a empty = End | Cons a (List a empty)

createEmptyList :: List a Empty
createEmptyList = End
{% endhighlight %}

And this works! By defining the type signature of `createEmptyList` we made sure that `empty` will
always assume the type of `Empty` when this function is called. Since `End` has a type signature
`End :: List a empty`, we just "casted" type param `empty` here into a specific type.

Let's try to do the same for the other data constructor, adding an element to the list:

{% highlight haskell %}
data Empty
data NonEmpty

data List a empty = End | Cons a (List a empty)

addElemToList :: a -> List a empty -> List a NonEmpty
addElemToList elem list = Cons elem list
{% endhighlight %}

What we are trying to achieve here is make sure that whenever an element is added to the list,
`empty` becomes `NonEmpty`, and we again use type signature for that, to provide that extra
information.
But if we try to compile this, we get the following error: `Couldn't match type ‘empty’ 
with ‘NonEmpty’`.

To understand the problem, let's remind ourselves of `Cons`'s type:

`Cons :: a -> List a empty -> List a empty.`

The problem is in that `Cons` requires `empty` to stay the same, so whatever type it is in the
input list, it must stay the same in the newly constructed list. Although we specified we want
to change it to `NonEmpty` in the `addElemToList`'s type signature, `Cons` is not flexible enough
to do that and this is why we got an error during compilation.

Although smart constructors might be a solution in some simpler cases (e.g. when we have "flat" data
and we are merely "casting" general type params into the specific ones, such as we did with `End`),
in this case where we have a recursive data structure it wasn't enough because the the initial
data constructor was too rigid.

### What is `List Int Double`?

Well, `List Int Double` means nothing, it doesn't make sense. We can only construct and know how
to work with lists whose `empty` type parameter is either `Empty` or `NonEmpty`.

But the problem is although it doesn't make sense, we can still write things like this and it will
happily compile:

{% highlight haskell %}
someListFn :: List a Bool -> Int
someListFn list = 23
{% endhighlight %}

There is no way to execute this function since there is no way to construct such a list where
`empty` type param is `Bool`, but strange stuff can appear in our codebase and we cannot detect
it in compile time.

Here is a more "real world" example when this could be a problem: let's say you are
using `Empty` and `NonEmpty` types for list as we explained above,
but you are also using `Yes` and `No` types for something else in your codebase. And then your
colleague starts implementing some new functionality for your lists, and by mistake he starts using
`Yes` and `No` in the place of `empty`. And there is nothing to stop him until he actually tries to
connect everything together and run the code!

The problem we see is there is no "safety" at the type level, we cannot say *"`empty` can be only
this kind of type"*. But, there is a mechanism that can help us.

### Not all types are used in the same way

Just a short observation before we continue. I wanted to put attention to the fact that we are now
differentiating between two possible uses of a type:
* type is used to *produce values* (store data) - e.g. `Int`, `Maybe Bool`, ...
* type is used only *at the type level as a designation of something*, never producing an
actual value - e.g. `Empty` and `NonEmpty`

Despite these very different uses, we currently don't have a way to differentiate between such
types - we declare them both in the same way and Haskell can't tell how are we going to use them
later.

### Data kinds

In standard Haskell each type has a *kind*, which can be thought of as a *"type of a type"*. E.g.:

{% highlight haskell %}
> :k Int
Int :: *                // Has values (e.g. 1, 2, 3, ...).

> :k Maybe
Maybe :: * -> *         // When given a value-producing type, has values.

> :k Either
Either :: * -> * -> *   // When given 2 value-producing types, has value.
{% endhighlight %}

And that is it, all kinds are expressed with `*`s and automatically derived for us. `*` means
*a type that has values*.

But as we saw earlier, this is not enough for us. We also want to cover that other use case so we
can say *"here goes only type(s) that tell us whether a list is empty or not."*

And this is exactly what `DataKinds` extension allows us to do, it lets us define other kinds
besides `*`:

{% highlight haskell %}
{-# LANGUAGE GADTs #-}
{-# LANGUAGE DataKinds #-}

data ListStatus = Empty | NonEmpty

data List a empty where
    End :: List a 'Empty 
    Cons :: a -> List a empty -> List a 'NonEmpty
{% endhighlight %}

Now let's see what happened here. We made a new type `ListStatus` and then we use its data
constructors (prefixed with `'`) in the place of types in GADT for `List`. Wut?

The thing with `DataKinds` is the following: for every type we create it additionaly creates for us
new types, named after data constructors we used and prefixed with `'`. It also creates a new kind,
which is named after the type's name. Specifically for this case:
* `DataKinds` created for us two extra types, `'Empty` and `'NonEmpty`
* Kinds of these new types are `ListStatus`
* These types cannot have values, they can be used only at the type level

I was really confused the first time I realized this. This extension just like that creates extra
types for us, without even asking us about it, for every type we create!

Since we are using only types of `ListStatus` kind in `List`'s data constructors' signatures,
Haskell inferred from that that `empty` type param must be of kind `ListStatus` and won't let us
use anythng else. If we try to create a function which takes `List a Bool`, we will receive the
following error:

`Expected kind ‘ListStatus’, but ‘Bool’ has kind ‘*’`

Which is exactly what we wanted! With this we achieved *kind safety*, besides the usual
*type safety* in the compile time.

To make things even more explicit, we can turn on `KindSignatures` extension which lets us
explicitly define kinds of type parameters in a type:

{% highlight haskell %}
{-# LANGUAGE GADTs #-}
{-# LANGUAGE DataKinds #-}
{-# LANGUAGE KindSignatures #-}

data ListStatus = Empty | NonEmpty

data List (a :: * ) (empty :: ListStatus) where
    End :: List a 'Empty 
    Cons :: a -> List a empty -> List a 'NonEmpty
{% endhighlight %}

Now everybody can see that `a` is a "standard" type that has values, while `empty` can be only
`'Empty` or `'NonEmpty`. We didn't have to write this explicit version as Haskell can infer it on
its own, but it is a matter of style and documentation. We can also omit `'` in front of types and
Haskell in a lot of cases can infer by itself if it is a type or data constructor. I found it easier
to have everything explicit for now, a lot is going on behind the scenes so this made it clearer
for me.

And that is it for this first part! We learned about type-level programming, how to use GADTs and
data kinds and saw everything together in action. Hope you found it useful, please let me know in
the comments if you have any questions, I said something wrong or I can explain something better.

In the Part 2 we will go even deeper and take a look at some more cool examples that build on top
of this one! Here's a teaser question: with our `List a empty` that we developed above, how would
you implement `safeTail` function which works only on non-empty lists,
analogous to what we have done with `safeHead`? Can you do it, what is its return type?
