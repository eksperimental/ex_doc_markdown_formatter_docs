# Inspect protocol
(Elixir v1.18.0-dev)

The `Inspect` protocol converts an Elixir data structure into an
algebra document.

This is typically done when you want to customize how your own
structs are inspected in logs and the terminal.

This documentation refers to implementing the `Inspect` protocol
for your own data structures. To learn more about using inspect,
see `Kernel.inspect/2` and `IO.inspect/2`.

## Inspect representation

There are typically three choices of inspect representation. In order
to understand them, let's imagine we have the following `User` struct:

    defmodule User do
      defstruct [:id, :name, :address]
    end

Our choices are:

1. Print the struct using Elixir's struct syntax, for example:
   `%User{address: "Earth", id: 13, name: "Jane"}`. This is the
   default representation and best choice if all struct fields
   are public.

2. Print using the `#User<...>` notation, for example: `#User<id: 13, name: "Jane", ...>`.
   This notation does not emit valid Elixir code and is typically
   used when the struct has private fields (for example, you may want
   to hide the field `:address` to redact person identifiable information).

3. Print the struct using the expression syntax, for example:
   `User.new(13, "Jane", "Earth")`. This assumes there is a `User.new/3`
   function. This option is mostly used as an alternative to option 2
   for representing custom data structures, such as `MapSet`, `Date.Range`,
   and others.

You can implement the Inspect protocol for your own structs while
adhering to the conventions above. Option 1 is the default representation
and you can quickly achieve option 2 by deriving the `Inspect` protocol.
For option 3, you need your custom implementation.

## Deriving

The `Inspect` protocol can be derived to customize the order of fields
(the default is alphabetical) and hide certain fields from structs,
so they don't show up in logs, inspects and similar. The latter is
especially useful for fields containing private information.

The supported options are:

- `:only` - only include the given fields when inspecting.

- `:except` - remove the given fields when inspecting.

- `:optional` - (since v1.14.0) do not include a field if it
  matches its default value. This can be used to simplify the
  struct representation at the cost of hiding information.

Whenever `:only` or `:except` are used to restrict fields,
the struct will be printed using the `#User<...>` notation,
as the struct can no longer be copy and pasted as valid Elixir
code. Let's see an example:

    defmodule User do
      @derive {Inspect, only: [:id, :name]}
      defstruct [:id, :name, :address]
    end
    
    inspect(%User{id: 1, name: "Jane", address: "Earth"})
    #=> #User<id: 1, name: "Jane", ...>

If you use only the `:optional` option, the struct will still be
printed as `%User{...}`.

## Custom implementation

You can also define your custom protocol implementation by
defining the `inspect/2` function. The function receives the
entity to be inspected followed by the inspecting options,
represented by the struct `Inspect.Opts`. Building of the
algebra document is done with `Inspect.Algebra`.

Many times, inspecting a structure can be implemented in function
of existing entities. For example, here is `MapSet`'s `inspect/2`
implementation:

    defimpl Inspect, for: MapSet do
      import Inspect.Algebra
    
      def inspect(map_set, opts) do
        concat(["MapSet.new(", Inspect.List.inspect(MapSet.to_list(map_set), opts), ")"])
      end
    end

The [`concat/1`](\`Inspect.Algebra.concat/1\`) function comes from
`Inspect.Algebra` and it concatenates algebra documents together.
In the example above it is concatenating the string `"MapSet.new("`,
the document returned by `Inspect.Algebra.to_doc/2`, and the final
string `")"`. Therefore, the MapSet with the numbers 1, 2, and 3
will be printed as:

    iex> MapSet.new([1, 2, 3], fn x -> x * 2 end)
    MapSet.new([2, 4, 6])

In other words, `MapSet`'s inspect representation returns an expression
that, when evaluated, builds the `MapSet` itself.

### Error handling

In case there is an error while your structure is being inspected,
Elixir will raise an `ArgumentError` error and will automatically fall back
to a raw representation for printing the structure. Furthermore, you
must be careful when debugging your own Inspect implementation, as calls
to `IO.inspect/2` or `dbg/1` may trigger an infinite loop (as in order to
inspect/debug the data structure, you must call `inspect` itself).

Here are some tips:

- For debugging, use `IO.inspect/2` with the `structs: false` option,
  which disables custom printing and avoids calling the Inspect
  implementation recursively

- To access the underlying error on your custom `Inspect` implementation,
  you may invoke the protocol directly. For example, we could invoke the
  `Inspect.MapSet` implementation above as:
  
      Inspect.MapSet.inspect(MapSet.new(), %Inspect.Opts{})

## Types

### t()

```elixir
@type t() :: term()
```

All the types that implement this protocol.

## Functions

### inspect(term, opts)

```elixir
@spec inspect(t(), Inspect.Opts.t()) :: Inspect.Algebra.t()
```

Converts `term` into an algebra document.

This function shouldn't be invoked directly, unless when implementing
a custom `inspect_fun` to be given to `Inspect.Opts`. Everywhere else,
`Inspect.Algebra.to_doc/2` should be preferred as it handles structs
and exceptions.



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
