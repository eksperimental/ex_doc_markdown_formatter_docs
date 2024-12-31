# MatchError exception
(Elixir v1.18.0-dev)

An exception raised when a pattern match (`=/2`) fails.

The following fields of this exception are public and can be accessed freely:

- `:term` (`t:term/0`) - the term that did not match the pattern

For example, this exception gets raised for code like this:

    [_ | _] = []




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
