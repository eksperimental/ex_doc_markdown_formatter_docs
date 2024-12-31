# Inspect.Opts 
(Elixir v1.18.0-dev)

Defines the options used by the `Inspect` protocol.

The following fields are available:

- `:base` - prints integers and binaries as `:binary`, `:octal`, `:decimal`,
  or `:hex`. Defaults to `:decimal`.

- `:binaries` - when `:as_binaries` all binaries will be printed in bit
  syntax.
  
  When `:as_strings` all binaries will be printed as strings, non-printable
  bytes will be escaped.
  
  When the default `:infer`, the binary will be printed as a string if `:base`
  is `:decimal` and if it is printable, otherwise in bit syntax. See
  `String.printable?/1` to learn when a string is printable.

- `:charlists` - when `:as_charlists` all lists will be printed as charlists,
  non-printable elements will be escaped.
  
  When `:as_lists` all lists will be printed as lists.
  
  When the default `:infer`, the list will be printed as a charlist if it
  is printable, otherwise as list. See `List.ascii_printable?/1` to learn
  when a charlist is printable.

- `:custom_options` (since v1.9.0) - a keyword list storing custom user-defined
  options. Useful when implementing the `Inspect` protocol for nested structs
  to pass the custom options through.
  
  It supports some pre-defined keys:
  
  - `:sort_maps` (since v1.14.4) - if set to `true`, sorts key-value pairs
    in maps. This can be helpful to make map inspection deterministic for
    testing, given maps key order is random.

- `:inspect_fun` (since v1.9.0) - a function to build algebra documents.
  Defaults to `Inspect.Opts.default_inspect_fun/0`.

- `:limit` - limits the number of items that are inspected for tuples,
  bitstrings, maps, lists and any other collection of items, with the exception of
  printable strings and printable charlists which use the `:printable_limit` option.
  If you don't want to limit the number of items to a particular number,
  use `:infinity`. It accepts a positive integer or `:infinity`.
  Defaults to `50`.

- `:pretty` - if set to `true` enables pretty printing. Defaults to `false`.

- `:printable_limit` - limits the number of characters that are inspected
  on printable strings and printable charlists. You can use `String.printable?/1`
  and `List.ascii_printable?/1` to check if a given string or charlist is
  printable. If you don't want to limit the number of characters to a particular
  number, use `:infinity`. It accepts a positive integer or `:infinity`.
  Defaults to `4096`.

- `:safe` - when `false`, failures while inspecting structs will be raised
  as errors instead of being wrapped in the `Inspect.Error` exception. This
  is useful when debugging failures and crashes for custom inspect
  implementations. Defaults to `true`.

- `:structs` - when `false`, structs are not formatted by the inspect
  protocol, they are instead printed as maps. Defaults to `true`.

- `:syntax_colors` - when set to a keyword list of colors the output is
  colorized. The keys are types and the values are the colors to use for
  each type (for example, `[number: :red, atom: :blue]`). Types can include
  `:atom`, `:binary`, `:boolean`, `:list`, `:map`, `:number`, `:regex`,
  `:string`, `:tuple`, or some types to represent AST like `:variable`,
  `:call`, and `:operator`.
  Custom data types may provide their own options.
  Colors can be any `t:IO.ANSI.ansidata/0` as accepted by `IO.ANSI.format/1`.
  A default list of colors can be retrieved from `IO.ANSI.syntax_colors/0`.

- `:width` - number of characters per line used when pretty is `true` or when
  printing to IO devices. Set to `0` to force each item to be printed on its
  own line. If you don't want to limit the number of items to a particular
  number, use `:infinity`. Defaults to `80`.


## Types

### color_key()

```elixir
@type color_key() :: atom()
```



### t()

```elixir
@type t() :: %Inspect.Opts{
  base: :decimal | :binary | :hex | :octal,
  binaries: :infer | :as_binaries | :as_strings,
  char_lists: term(),
  charlists: :infer | :as_lists | :as_charlists,
  custom_options: keyword(),
  inspect_fun: (any(), t() -&gt; Inspect.Algebra.t()),
  limit: non_neg_integer() | :infinity,
  pretty: boolean(),
  printable_limit: non_neg_integer() | :infinity,
  safe: boolean(),
  structs: boolean(),
  syntax_colors: [{color_key(), IO.ANSI.ansidata()}],
  width: non_neg_integer() | :infinity
}
```



## Functions

### default_inspect_fun()
*(since 1.13.0)* 
```elixir
@spec default_inspect_fun() :: (term(), t() -&gt; Inspect.Algebra.t())
```

Returns the default inspect function.


### default_inspect_fun(fun)
*(since 1.13.0)* 
```elixir
@spec default_inspect_fun((term(), t() -&gt; Inspect.Algebra.t())) :: :ok
```

Sets the default inspect function.

Set this option with care as it will change how all values
in the system are inspected. The main use of this functionality
is to provide an entry point to filter inspected values,
in order for entities to comply with rules and legislations
on data security and data privacy.

It is **extremely discouraged** for libraries to set their own
function as this must be controlled by applications. Libraries
should instead define their own structs with custom inspect
implementations. If a library must change the default inspect
function, then it is best to ask users of your library to explicitly
call `default_inspect_fun/1` with your function of choice.

The default is `Inspect.inspect/2`.

#### Examples

    previous_fun = Inspect.Opts.default_inspect_fun()
    
    Inspect.Opts.default_inspect_fun(fn
      %{address: _} = map, opts ->
        previous_fun.(%{map | address: "[REDACTED]"}, opts)
    
      value, opts ->
        previous_fun.(value, opts)
    end)


### new(opts)
*(since 1.13.0)* 
```elixir
@spec new(keyword()) :: t()
```

Builds an `Inspect.Opts` struct.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
