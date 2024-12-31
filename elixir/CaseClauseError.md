# CaseClauseError exception
(Elixir v1.18.0-dev)

An exception raised when a term in a `case/2` expression
does not match any of the defined `->` clauses.

The following fields of this exception are public and can be accessed freely:

- `:term` (`t:term/0`) - the term that did not match any of the clauses

For example, this exception gets raised for a `case/2` like the following:

    case System.unique_integer() do
      bin when is_binary(bin) -> :oops
      :ok -> :neither_this_one
    end



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
