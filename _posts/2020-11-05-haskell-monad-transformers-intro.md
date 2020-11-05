---
layout: post
title: "Intro to monad transformers in Haskell"
---

*NOTE: this tutorial assumes you are already familiar with monads and how they work. If not,
I would recommend learning about monads first.*

## Why monad transformers?

Let's say we have functions with the following signatures:

{% highlight haskell %}
type UserId = String

fetchUsersName :: UserId -> IO (Maybe String)

fetchUsersSurname :: UserId -> IO (Maybe String)

fetchUsersAge :: UserId -> IO (Maybe Int)
{% endhighlight %}

Let's say we are building an app and this is an API we are given to access specific user's data.
These functions take user's id and go into the world to perform an IO action - e.g. a
network request, or read from the disk. Also, an IO action can fail if e.g.
there was no network connection available - that is why `Maybe` is in the return type.

With that, we want to implement the following function:

{% highlight haskell %}
fetchUsersData :: UserId -> IO (Maybe (String, String, Int))
{% endhighlight %}

This function tries to fetch all the data for the specific user (name, surname and age) and if
any piece of this data couldn't be obtained it will declare itself as failed by returning
`Nothing` within IO monad.

This is how we would go about implementing `fetchUsersData`:

{% highlight haskell %}
fetchUsersData :: UserId -> IO (Maybe (String, String, Int))
fetchUsersData userId = do
    maybeName <- fetchUsersName userId
    case maybeName of
        Nothing -> return Nothing
        Just name -> do
            maybeSurname <- fetchUsersSurname userId
            case maybeSurname of
                Nothing -> return Nothing
                Just surname -> do
                    maybeAge <- fetchUsersAge userId
                    case maybeAge of
                        Nothing -> return Nothing
                        Just age -> return (Just (name, surname, age))
{% endhighlight %}

And this would work! The only not so nice thing is that we have this "staircasing" typical for
nesting multiple `Maybe`s. We would normally solve this by running things within a `Maybe` monad,
but this time we are in `IO` monad so we can't do that!

To generalize the problem, it occurs when we have one monad within
another (`Maybe` within `IO` here) and would like to access "powers" of the inner monad to make
our code nicer.

This is a common case when using monads in Haskell and exactly what monad transformers are set to
solve - making it possible to user "powers" of one monad within another monad. In this case, we
would like to extend `IO`'s capabilities with `Maybe`'s power to exit the computation early.

## What is a monad transformer?

Here is what the [documentation](http://hackage.haskell.org/package/transformers-0.5.6.2/docs/Control-Monad-Trans-Class.html) says:

*A monad transformer makes a new monad out of an existing monad, such that computations of
the old monad may be embedded in the new one.*

And then there is a typeclass `MonadTrans` which monad transfomer has to implement, which means
monad transformer is actually a type. Monad transformer is a monad as well.

Here is how it looks like for our case, where we want to extend `IO` with `Maybe` capabilities:

{% highlight haskell %}
newtype MaybeT m a = MaybeT { runMaybeT :: m (Maybe a) }
{% endhighlight %}

