---
layout: post
title: "Intro to monad transformers in Haskell"
---

*NOTE: this tutorial assumes you are already familiar with monads. If not,
I would recommend learning about monads first.*

## Why monad transformers?

Let's say we have functions with the following signatures:

{% highlight haskell %}
type UserId = String

fetchUsersName :: UserId -> IO (Maybe String)

fetchUsersSurname :: UserId -> IO (Maybe String)

fetchUsersAge :: UserId -> IO (Maybe Int)
{% endhighlight %}

Imagine we are building an app and this is an API we are given to access specific user's data.
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

And this would work! The only not so nice thing is that we have this "staircasing" typical when
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

And then there is a typeclass `MonadTrans` which every monad transfomer has to implement, which means
monad transformer is actually a type. Monad transformer is a monad as well.

Here is how it looks like in our case, where we want to extend `IO` with `Maybe` capabilities:

{% highlight haskell %}
newtype MaybeT m a = MaybeT { runMaybeT :: m (Maybe a) }
{% endhighlight %}

`MaybeT m a` is a monad transformer version of `Maybe a`. There is only that extra `m` which stands
for an "inner" monad that will be expanded with `Maybe` capabilities.

Regarding the named field of `MaybeT` (`runMaybeT`), here's another bit from the documentation:

*Each monad transformer also comes with an operation `runXXXT` to unwrap the transformer, exposing a computation of the inner monad.*

If we wrapped `IO` within `MaybeT` and then couldn't get it back again that would be a problem,
since the function we are implementing needs to return `IO (Maybe (String, String, Int))`.

From this we can generalize and conclude that a monad transformer has:
* one additional type parameter (compared to its monad counterpart) - an inner monad that is being
expanded with capabilities of the outer monad
* `runXXXT` function/field which returns the original, "inner" monad

## What does it mean "monads don't compose"?

If you have been learning about monad transformers, odds are you came across this statement. I did
too, and often it was the first thing discussed when talking about monad transformers. But I
couldn't understand what does it mean to "compose" monads nor why it is a problem if that cannot be
done.

As you also probably know and as we saw in the example with `Maybe`, each monad has its monad
transformer counterpart (`Maybe` and `MaybeT`, `Either` and `EitherT`, ...). Each of these monad
transformer counterparts had to be manually and separately implemented by somebody.

That is exactly what the statement in question (monad composition) challenges - why do we have to
do so much work, do we really need to implement `xxxT` version of each monad? It would be awesome
if we could somehow automate this.

And that is where composing monads would come in handy. Monad composition is creating a new type
which is parametrized by (any) two monads and is then also itself a monad. It would look like this:

{% highlight haskell %}
newtype MonadComposition m1 m2 a = MonadComposition { getMC :: m1 (m2 a)) }
{% endhighlight %}

This is the general case of the example above, where `m1` and `m2` were `IO` and `Maybe`,
respectively (one monad wrapped in another).

Now the main question here is can we implement the following:
{% highlight haskell %}
instance (Monad m1, Monad m2) => Monad (MonadComposition m1 m2) where
    return = ...
    join = ...
{% endhighlight %}

If we could, that would mean `MonadComposition m1 m2` is also a monad. Meaning that we solved
the general case of composing any two monads! If that was true, we wouldn't need to bother
with implementing `MaybeT`, `EitherT`, `ReaderT` separately - `MonadComposition` would cover all
of that for us automatically!

But as you reckon, that is not the case. It is not possible to make `MonadComposition m1 m2` an
instance of `Monad` typeclass. It can be proved and it is not trivial. From the intuitive
perspective, we can understand that we need more information, the general case is not covering it.
If the "outer" monad is `Maybe`, we need to implement what `MaybeT` will do in terms of `Nothing`
and `Just x`, which means we need the specifics.

So that is it! Now you know what "monads don't compose" means and why it is important.

## MaybeT

We used it as an example in the introduction, but let's now officially take look at it.

From the [docs](http://hackage.haskell.org/package/transformers-0.5.6.2/docs/Control-Monad-Trans-Maybe.html):

*The `MaybeT` monad transformer extends a monad with the ability to exit the computation without returning a value.*

*A sequence of actions produces a value only if all the actions in the sequence do. If one exits, the rest of the sequence is skipped and the composite action exits.*

Here is the type definition:

{% highlight haskell %}
newtype MaybeT m a = MaybeT { runMaybeT :: m (Maybe a) }
{% endhighlight %}

`m` being an arbitrary monad composed with `Maybe` monad. Let's now see it in action with our example from the beginning:
{% highlight haskell %}
fetchUsersData :: UserId -> IO (Maybe (String, String, Int))
fetchUsersData userId = runMaybeT $ do
    name <- MaybeT $ fetchUsersName userId
    surname <- MaybeT $ fetchUsersSurname userId
    age <- MaybeT $ fetchUsersAge userId

    return (name, surname, age)
{% endhighlight %}

We can see from the types this works. We don't end on the left hand side of `<-` anymore with `Maybe a` which we then have to
unpack (leading to the staircasing we saw before), but we get `a` directly.

But what actually happens behind the curtains? Let's see how `MaybeT` implements `Monad` typeclass:

{% highlight haskell %}
instance Monad (MaybeT m) where
    -- yields a computation that produces value
    return :: a -> MaybeT m a
    return = MaybeT . return . Just

    -- if a computation within monad failed, short-circuits to failed.
    (>>=) :: MaybeT m a -> (a -> MaybeT m b) -> MaybeT m b
    x >>= f = MaybeT $ do -- entering inner monad m
        v <- runMaybeT x  -- unpacking x from `MaybeT m a` to `Maybe a`
        case v of
            Nothing -> return Nothing
            Just y -> runMaybeT (f y)
{% endhighlight %}

We can see that actually here `MaybeT` does the heavy lifting for us, what we previously did
manually (staircasing example) - it operates within `m` (`IO` in the example) and gets to `Maybe a`
and then does that "manual" check whether it is `Nothing` or not.

## Lifting

We said monad transformers are all about combining the powers of two or more monads. With our
`MaybeT IO a` example we've shown how we can get `Maybe`'s power and make sure that the whole computation is
short-circuited to `Nothing` if some sub-computation failed. But what if we wanted to do
some IO stuff, e.g. print `Fetching data...`?

This is why we have `lift`, which is the only function of `MonadTrans` typeclass:

{% highlight haskell %}
class MonadTrans t where
    -- | Lift a computation from the argument monad to the constructed monad.
    lift :: (Monad m) => m a -> t m a
{% endhighlight %}

Argument monad is the "inner" monad (`IO` in our example) and constructed monad is the actual
monad transformer (`MaybeT` in our case).

So this is the function that allows us to "lift" the inner monad's computation
into the monad transformer's realm, hence the name. It allows us to do this:

{% highlight haskell %}
fetchUsersData :: UserId -> IO (Maybe (String, String, Int))
fetchUsersData userId = runMaybeT $ do
    lift $ putStrLn "Fetching data..." -- <- NEW

    name <- MaybeT $ fetchUsersName userId
    surname <- MaybeT $ fetchUsersSurname userId
    age <- MaybeT $ fetchUsersAge userId

    return (name, surname, age)
{% endhighlight %}

`lift` here does exactly what it says in its signature, takes `IO ()` and lifts it into `MaybeT IO ()` so we can
call this printing action within `MaybeT` monad.

It is also maybe interesting to see how `MaybeT` implements `lift`:

{% highlight haskell %}
instance MonadTrans MaybeT where
    lift = MaybeT . liftM Just
{% endhighlight %}

In the case of printing from above, `IO ()` would come in, `liftM Just` would produce `IO (Just ())` and then `MaybeT` data constructor would create an instance of `MaybeT IO ()` type.

Phew, we just went through our first monad transformer! Let's now take a look at another one - `ExceptT`.

## ExceptT

`ExceptT` is a monad transformer version of `Either` and is very similar to `MaybeT`, just as `Either` is similar to `Maybe`.
The only difference is that in the case of the failure the exception that is thrown actually contains a value (additional error data) rather than just `Nothing`.

## ReaderT

As we know, `Reader` monad is useful
