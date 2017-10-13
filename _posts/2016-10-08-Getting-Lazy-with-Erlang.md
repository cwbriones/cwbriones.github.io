---
layout: post
date: 2016-10-08 21:32:57
title: Getting Lazy with Erlang
published: true
---

I've been playing around with Haskell again, and it's amazing how easily it expresses recursive concepts. [^1] For the uninitiated, Haskell has a list data type that is _lazily_ evaluated. That is, elements of the list are only evaluated if they're needed to continue the computation. This allows you to do all sorts of neat things like create an infinite list of integers or build a list that refers to itself.

```haskell
-- Create an infinite list of integers
integers = [1..]

-- Filter out the odd values
isOdd = (> 0) . (flip mod 2)

odds = filter isOdd integers

-- Map them to squares
oddSquares = map (^2) odds

-- Take the first 10 from this list.
-- This is when the list actually gets evaluated.
--
-- firstTen = [1,9,25,49,81,121,169,225,289,361]
firstTen = take 10 oddSquares
```

In my day-to-day I'm programming Erlang, which doesn't have the concept of lazy-evaluation baked right in from the start. In Erlang lists are strictly (or eagerly) evaluated, so the above breaks down pretty much right away.

```erlang
%% Create an infinite list of integers
%% integers() ->
%%    ???
```

We can't even express the original list of integers with erlang's built-in lists. So how can we
get something like what we have in Haskell?

## Funs as contexts

If we look around, it's quickly apparent that functions do this by their very nature - they let you pass around blocks of code as data. Wrapping a value in a lambda only evaluates that value when the lambda is called:

```
1> io:format("Hey!\n").
Hey!
ok
2> F = fun() -> io:format("Hey!"\n) end.
#Fun<erl_eval.20.50752066>
3> F().
Hey!
ok
```

We can use this to build our lazy lists in Erlang, we'll call them *streams*. [^2]

You can think of streams as particular types of functions. They need to return the next value in the stream, if any, otherwise tell us we've reached the end. This leads to a elegantly recursive typespec:

```erlang
-type stream(A) :: fun(() -> halt | {A, stream(A)}.
```

If the stream returns `halt`, we know we've reached the end. Otherwise we get our next value along with a new stream for continuing iteration.

## Building our first stream

First of all, let's wrap up actual function calls to our stream with the function `yield/1'.

```erlang
-spec yield(Stream :: stream(A)) -> halt | {A, stream(A)}.
yield(F) when is_function(F) -> F().
```

This may not seem immediately useful, but it will come into play later when we want to interop with built-in types. Now let's get to work! We'll begin by creating our list of integers.

```erlang

-spec naturals() -> stream(integer()).
naturals() -> naturals(0).

-spec naturals(integer()) -> stream(integer()).
naturals(N)
  fun() ->
    {N, naturals(N + 1)}
  end.
```

Easy! The stream of natural numbers starting at `N` is followed by the stream starting at `N+1`. Notice that our `naturals/1` stream unconditionally refers to itself, it's __infinite__. Since our recursive call is wrapped up neatly in a `fun`, we can express this definition without fear of an infinite loop. We can check that it works as expected in the repl.

```
1> c(streams).
{ok, streams}
2> Ns = naturals().
#Fun<streams.2.68943458>
3> {_, Ns2} = yield(Ns).
{0, #Fun<streams.2.68943458>}
4> {_, Ns3} = yield(Ns2).
{1, #Fun<streams.2.68943458>}
5> {_, Ns4} = yield(Ns3).
{2, #Fun<streams.2.68943458>}
```

As you can see on the right-hand side, repeated calls to yield give us back `0, 1, 2` as expected.

## Stream Transformers

Now that we have our infinite stream of integers, we need to be able to transform this stream in a list-like fashion. Erlang provides higher-order functions such as `lists:map/2` and `lists:filter/2` to manipulate and transform its lists. We'll define the same for streams.

Implementing `map/2` is pretty straightforward. Like before we can express map quite nicely with a recursive call. If we have elements in the original stream, we immediately apply `Fun` to the next element while returning the mapped "tail", otherwise we halt.

```erlang
map(Fun, Stream) ->
  fun() ->
    case yield(Stream) of
      {X, Xs} -> {Fun(X), map(Fun, Xs)};
      halt -> halt
    end
  end.
```

`filter/2` is a bit trickier, can you think of the reason?

```erlang
filter(Pred, Stream) ->
  fun() -> do_filter(Pred, Stream) end.

do_filter(Pred, Stream) ->
  case yield(Stream) of
    {X, Xs} ->
      case Pred(X) of
        true -> {X, filter(Fun, Xs)};
        false -> do_filter(Fun, Xs)
      end;
    halt -> halt
  end.
```

Unlike `map`, filter can contain less elements than the original stream, which means we need a way of _skipping_ elements. To get around this, we define a helper `do_filter` as the eagerly-evaluated innards of our lazy stream. If the predicate function returns true, everything is fine and dandy and we can recursively call filter like before.

If it's false, however, we need to call `do_filter` which will evaluate __immediately__. This should be expected - after all, the whole point of lazy evaluation is to only perform computation when needed, and we need to continue to the next element.

## Forcing evalution

Now that we can define and transform our stream, we need to be able to evaluate it. Otherwise it wouldn't serve much use, would it? We'll define a helper, `to_list/1` that repeated `yield`s elements into a list until we reach the end of the stream:

```erlang
-spec to_list(stream(A)) -> list(A).
to_list(Stream) -> to_list(Stream, []).

to_list(Stream, Acc) ->
  case yield(Stream) of
    {X, Xs} -> to_list(Xs, [X|Acc])
    halt -> lists:reverse(Acc);
  end.
```

This alone isn't enough to perform our computation. Up until now our stream has been infinite. If we called `to_list` on our stream now, we'd continue `yield`ing elements until we ran out of memory:

```
1> streams:to_list(streams:naturals()).
(Hangs forever...)
```

So let's define `take/2`

```erlang
-spec take(integer(), stream(A)) -> stream(A).
take(0, Stream) -> halt;
take(N, Stream) when N >= 0 ->
  fun() ->
    case yield(Stream) of
      {X, Xs} -> {X, take(N - 1, Xs)};
      halt -> halt
    end
  end.
```

While `take/2` still recursively returns itself as a stream, this time there's a base case in the recursion. If we're taking `0` elements, then there isn't anything to take at all! With that out of the way, we can finally replicate the Haskell example from earlier using our helper functions and a little squinting:

```erlang
Integers = streams:naturals(),
IsOdd = fun(N) -> mod(N, 2) > 0 end,
Odds = streams:filter(IsOdd, streams:naturals()),
OddSquares = streams:map(fun(N) -> N * N end, Odds),
Take10 = streams:take(10, OddSquares),
streams:to_list(Take10).
%% > [1,9,25,49,81,121,169,225,289,361]
```

Neat!

## Yielding from lists

As a bonus, we can easily add a case to `yield/1` that lets us use streams and lists interchangeably.

```erlang
-spec yield(Stream :: stream(A)) -> halt | {A, stream(A)}.
yield(F) when is_function(F) -> F().
yield([X|Xs]) -> {X, Xs};
yield([]) -> halt.
```

In this way, we can think of lists as their own type of stream. The indirection through yield acts as a simple interface - by adding additional function heads we can easily extend the notion of a stream to any iterable structure. This is very powerful, as it lets us mix-and-match these structures whenever we want. [^3]

## Until next time

We've seen that while infinite streams are certainly nice to have as a first-class feature, they're easily expressible through the power of simple functions. There are more complex examples of stream transformers and combinators available, but I'll leave those for another time. If you're curious or want to play around with them right now, all of the above (and more!) is available [on my GitHub](https://github.com/cwbriones/streams).

Thanks for reading!

[^1]: I'd highly recommend it if you haven't dealt with anything similar - it changes your way of thinking. Here's a [starting point.](http://learnyouahaskell.com/chapters)

[^2]: Those of you more familiar with the erlang ecosystem will know that I shamelessly took this from the excellent [Stream](http://elixir-lang.org/docs/stable/elixir/Stream.html) module found in Elixir.

[^3]: This idea is used to great effect in Elixir with its [Enumerable](http://elixir-lang.org/docs/stable/elixir/Enumerable.html) protocol, it fills a longstanding erlang sore spot.
