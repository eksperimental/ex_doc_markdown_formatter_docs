# List.Chars protocol
(Elixir v1.18.0-dev)

The `List.Chars` protocol is responsible for
converting a structure to a charlist (only if applicable).

The only function that must be implemented is
`to_charlist/1` which does the conversion.

The `to_charlist/1` function automatically imported
by `Kernel` invokes this protocol.

## Types

### t()

```elixir
@type t() :: term()
```

All the types that implement this protocol.

## Functions

### to_charlist(term)

```elixir
@spec to_charlist(t()) :: charlist()
```

Converts `term` to a charlist.



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
