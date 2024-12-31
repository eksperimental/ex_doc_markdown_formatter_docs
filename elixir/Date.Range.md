# Date.Range 
(Elixir v1.18.0-dev)

Returns an inclusive range between dates.

Ranges must be created with the `Date.range/2` or `Date.range/3` function.

The following fields are public:

- `:first` - the initial date on the range
- `:last` - the last date on the range
- `:step` - (since v1.12.0) the step

The remaining fields are private and should not be accessed.

## Types

### t()

```elixir
@type t() :: %Date.Range{
  first: Date.t(),
  first_in_iso_days: days(),
  last: Date.t(),
  last_in_iso_days: days(),
  step: pos_integer() | neg_integer()
}
```





---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
