# WithClauseError exception
(Elixir v1.18.0-dev)

An exception raised when a term in a `with/1` expression
does not match any of the defined `->` clauses in its `else`.

The following fields of this exception are public and can be accessed freely:

- `:term` (`t:term/0`) - the term that did not match any of the clauses

For example, this exception gets raised for a `with/1` like the following, because
the `{:ok, 2}` term does not match the `:error` or `{:error, _}` clauses in the
`else`:

    with {:ok, 1} <- {:ok, 2} do
      :woah
    else
      :error -> :error
      {:error, _} -> :error
    end



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
