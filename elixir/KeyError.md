# KeyError exception
(Elixir v1.18.0-dev)

An exception raised when a key is not found in a data structure.

For example, this is raised by `Map.fetch!/2` when the given key
cannot be found in the given map.

The following fields of this exception are public and can be accessed freely:

- `:term` (`t:term/0`) - the data structure that was searched
- `:key` (`t:term/0`) - the key that was not found




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
