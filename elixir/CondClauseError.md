# CondClauseError exception
(Elixir v1.18.0-dev)

An exception raised when no clauses in a `cond/1` expression evaluate to a truthy value.

For example, this exception gets raised for a `cond/1` like the following:

    cond do
      1 + 1 == 3 -> :woah
      nil -> "yeah this won't happen
    end




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
