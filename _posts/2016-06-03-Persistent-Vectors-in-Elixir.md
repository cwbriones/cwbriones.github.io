---
layout: post
date: 2016-06-03 01:34:53
title: Persistent Vectors in Elixir
published: false
---

I really enjoy working with Elixir; it's very easy to be productive with it. The whole thing _feels_ modern and well-designed every step of the way through, from the tooling to the standard library. After working in Erlang most days, it's like you've taken a rag and wiped away 30 years of legacy cruft from what underneath is a brilliant platform. With all that said it's no surprise that I've been reaching for it more and more when working on projects in my spare time.

## Persistent Structures

One of the tenets of Elixir is that it is *functional*. You handle immutable data that is isolated among the many processes running in your application. That makes it easier to reason about and apply these levels of massive concurrency the OTP platform is known for. Along with it, however, are the loss of low-level handling of data structures.

Because everything is immutable, objects need to be copied between functions and processes as they are sent around and altered. If done naively, this can be _expensive and painful_. We would end up copying every portion of our data at every moment of alteration and our problem would quickly turn into a sluggish mess. Of course, this isn't what happens. Instead, these immutable datastructures are designed with sharing in mind. We leverage the design of our structures to only copy what is necessary.

Erlang and Elixir do exactly this with its core data structures. When you're adding an element to the head of the list, the entire list isn't copied. Instead what happens is we form a new cell with a _reference_ to the old list as its tail, that way we only need to allocate as much as is necessary to specify the _changes_ to the structure that define our new one. The same goes for maps, although the details are a big more complicated.

## Vectors

One of the most commonly-used structures is the dynamic array, or *vector* [^1]. They're often very useful when working on problems mutating a large set of data. We can grow the vector as needed and we still get constant-time access and mutation guarantees, so it's no surprise that they are so useful.

Back in functional land, we're much more likely to see the _linked-list_ as our collection object. As mentioned earlier, the structure of a list allows us to efficiently share immutable portions of it as we alter it. Using it isn't usually a problem, we can get incredibly far with higher-order transformations of our list. It becomes a problem, however, when our problem domain requires us to access elements of the list at random, or mutate an item in the list. To do either of these would require traversal of the list. We can't just pull a value out and put a new one in - we'd have to go across the entire thing. While we save on allocating new tails, every alteration still requires us to allocate new cells for all the elements preceding the mutated position.

## The deceptively complicated `:array`

This clearly isn't desirable. What we really need is something like the vector of the imperative world, allowing for constant access lookup and mutation.

Erlang has the `:array` library for those times when you need these guarantees about your data. The thing is though, `:array` is quite complicated. They're implemented using what is known as judy arrays[^2], and if you jump to wikipedia, you'll see this fun line:

> Judy arrays are extremely complicated. The smallest implementations are thousands of lines of code.

That's no exaggeration. The [array implementation](https://github.com/erlang/otp/blob/maint/lib/stdlib/src/array.erl) clocks in at nearly 2KLOC - not exactly the pinnacle of readability.


[^1]: This name will be most familiar to you C++ or Rust devs out there.
[^2]: If you're interested in reading more check out [Wikipedia](https://en.wikipedia.org/wiki/Judy_array).
