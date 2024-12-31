# IO.Stream 
(Elixir v1.18.0-dev)

Defines an `IO.Stream` struct returned by `IO.stream/2` and `IO.binstream/2`.

The following fields are public:

- `device`        - the IO device
- `raw`           - a boolean indicating if bin functions should be used
- `line_or_bytes` - if reading should read lines or a given number of bytes

It is worth noting that an IO stream has side effects and every time you go
over the stream you may get different results.


## Types

### t()

```elixir
@type t() :: %IO.Stream{
  device: IO.device(),
  line_or_bytes: :line | non_neg_integer(),
  raw: boolean()
}
```





---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
