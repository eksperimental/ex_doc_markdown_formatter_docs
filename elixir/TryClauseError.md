# TryClauseError exception
(Elixir v1.18.0-dev)

An exception raised when a term in a `try/1` expression
does not match any of the defined `->` clauses in its `else`.

The following fields of this exception are public and can be accessed freely:

- `:term` (`t:term/0`) - the term that did not match any of the clauses



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
