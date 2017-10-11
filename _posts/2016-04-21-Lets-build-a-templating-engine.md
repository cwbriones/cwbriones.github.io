---
layout: post
date: 2016-04-21 17:56:44
title: Building a Templating Engine - Tokenization
permalink: posts/templating-engine-part-1
published: false
---

Lately, I've found myself reaching for Elixir more and more in my personal projects. It's powerful, expressive, and it's beautiful. Of course, these qualities don't matter if you can't build something of value with it, so I should do just that and make something useful - in this case, a templating engine.

Of course, those of you familiar with elixir may be thinking, why not use EEx? It comes packaged with Elixir and is well-tested in Phoenix applications. That may be true, but it embeds too much logic in the template for my tastes. I'd prefer something simpler and stripped down. That's why we'll be writing an implementation of Mustache.

## But there are already Mustache libraries!

Why? Because it's small, simple, and has a very thorough specification. There are already several libraries out there for mustache in Elixir, but none quite reach feature parity with what EEx offers. A good library would be able to

* Render templates from a file or string.
* Precompile templates for faster execution.
* Precompile templates directly into an Elixir AST for embedding.

## Let's break this down.

It's important both to focus on what you want to support and how you'll go about doing this, so we can break this implementation up into several logical chunks.

1. Tokenization of the template.
2. Parsing tokens into an AST.
3. Compiling the AST into an executable format.
4. Translating compilation for embedding.

As you can guess, we'll focus on the first task for this post, and leave the rest for later. So let's jump right on in!

## Getting Started

Let's start by creating a new mix application with `mix new stache`.

```
Christian λ mix new stache
* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/stache.ex
* creating test
* creating test/test_helper.exs
* creating test/stache_test.exs

Your Mix project was created successfully.
You can use "mix" to compile it, test it, and more:

    cd stache
    mix test

Run "mix help" for more commands.
```

Awesome! `mix new` generates a scaffold for our project so we can jump right in. Let's start by modeling our problem and translating that into a specification we can meet. Looking at the mustache spec, we can see several valid cases we need to support.

* Plaintext
* Simple interpolation `{{ "{{simple" }}}}`
* Escaped interpolation `{{ "{{{escaped" }}}}}`
* Beginning a section: `{{ "{{#begin" }}}}`
* Beginning an inverted section: `{{ "{{^begin" }}}}`
* Ending a section: `{{ "{{/begin" }}}}`
* Writing a comment `{{ "{{! This is a comment" }}}}`
* Including a partial `{{ "{{> my_partial" }}}}`

It's not important for you to know what each of these does in detail at the moment, we can focus on that logic later after we handle tokenization. We can distinguish between these during compilation by assigning them a tag with their contents.

```elixir
%{ # Input text         Extracted Token
  "elixir is great"  => {:text, "elixir is great"}
  "{{ "{{simple" }}}}"       => {:double, "simple"}
  "{{ "{{{escaped" }}}}}"    => {:triple, "escaped"}
  "{{ "{{#begin" }}}}"       => {:begin, "begin"}
  "{{ "{{^begin" }}}}"       => {:begin_inverted, "begin"}
  "{{ "{{/begin" }}}}"       => {:end, "begin"}
  "{{ "{{> my_partial" }}}}" => {:partial, "my_partial"}
  "{{ "{{! comment" }}}}"    => {:comment, "comment"}
}
```

We can use these examples can be our test cases.

## Text and Interpolated forms

These are the easiest cases, since most characters will be part of a plain-text form and interpolation markers will form the basis for other forms.

Our tests are pretty simple

```elixir
# test/tokenizer_test.exs
defmodule Tokenizer do
  use ExUnit.Case

  import Stache.Tokenizer

  test "plain text input" do
    assert tokenize("This is some text") == [{:text, "This is some text"}]
  end

  test "basic interpolation" do
    assert tokenize("{{foobar}}") == [{:double, "foobar"}]
  end

  test "escaped interpolation" do
    assert tokenize("{{{escaped}}}") == [{:triple, "escaped"}]
  end
end
```

If we run these now we see that they all fail (since we haven't defined `tokenize`.

```elixir
defmodule Stache.Tokenizer do
  def tokenize(template) do
    template
    |> String.to_codepoints
    |> tokenize([], :text, [])
  end

  def tokenize([], tokens, :text, _buffer), do: Enum.reverse(tokens)
  def tokenize(["{", "{", "{" | stream], tokens, :text, buffer) do
  end
  def tokenize(["}", "}", "}" | stream], tokens, :text, buffer) do
  end
  def tokenize(["{", "{" | stream], tokens, :text, buffer) do
  end
  def tokenize(["}", "}" | stream], tokens, :text, buffer) do
  end
end
```
