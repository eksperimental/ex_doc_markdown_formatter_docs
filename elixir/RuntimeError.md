# RuntimeError exception
(Elixir v1.18.0-dev)

An exception for a generic runtime error.

This is the exception that `raise/1` raises when you pass it only a string as
a message:

    iex> raise "oops!"
    ** (RuntimeError) oops!

You should use this exceptions sparingly, since most of the time it might be
better to define your own exceptions specific to your application or library.
Sometimes, however, there are situations in which you don't expect a condition to
happen, but you want to give a meaningful error message if it does. In those cases,
`RuntimeError` can be a good choice.

## Fields

`RuntimeError` exceptions have a single field, `:message` (a `t:String.t/0`),
which is public and can be accessed freely when reading or creating `RuntimeError`
exceptions.



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
