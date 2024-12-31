# File.Stream 
(Elixir v1.18.0-dev)

Defines a `File.Stream` struct returned by `File.stream!/3`.

The following fields are public:

- `path`          - the file path
- `modes`         - the file modes
- `raw`           - a boolean indicating if bin functions should be used
- `line_or_bytes` - if reading should read lines or a given number of bytes
- `node`          - the node the file belongs to

## Types

### t()

```elixir
@type t() :: %File.Stream{
  line_or_bytes: term(),
  modes: term(),
  node: term(),
  path: term(),
  raw: term()
}
```





---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
