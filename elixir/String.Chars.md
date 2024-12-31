# String.Chars protocol
(Elixir v1.18.0-dev)

The `String.Chars` protocol is responsible for
converting a structure to a binary (only if applicable).

The only function required to be implemented is
`to_string/1`, which does the conversion.

The `to_string/1` function automatically imported
by `Kernel` invokes this protocol. String
interpolation also invokes `to_string/1` in its
arguments. For example, `"foo#{bar}"` is the same
as `"foo" <> to_string(bar)`.

## Types

### t()

```elixir
@type t() :: term()
```

All the types that implement this protocol.

## Functions

### to_string(term)

```elixir
@spec to_string(t()) :: String.t()
```

Converts `term` to a string.



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
