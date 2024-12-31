# Protocol behaviour
(Elixir v1.18.0-dev)

Reference and functions for working with protocols.

A protocol specifies an API that should be defined by its
implementations. A protocol is defined with `Kernel.defprotocol/2`
and its implementations with `Kernel.defimpl/3`.

## Example

In Elixir, we have two nouns for checking how many items there
are in a data structure: `length` and `size`.  `length` means the
information must be computed. For example, `length(list)` needs to
traverse the whole list to calculate its length. On the other hand,
`tuple_size(tuple)` and `byte_size(binary)` do not depend on the
tuple and binary size as the size information is precomputed in
the data structure.

Although Elixir includes specific functions such as `tuple_size`,
`binary_size` and `map_size`, sometimes we want to be able to
retrieve the size of a data structure regardless of its type.
In Elixir we can write polymorphic code, i.e. code that works
with different shapes/types, by using protocols. A size protocol
could be implemented as follows:

    defprotocol Size do
      @doc "Calculates the size (and not the length!) of a data structure"
      def size(data)
    end

Now that the protocol can be implemented for every data structure
the protocol may have a compliant implementation for:

    defimpl Size, for: BitString do
      def size(binary), do: byte_size(binary)
    end
    
    defimpl Size, for: Map do
      def size(map), do: map_size(map)
    end
    
    defimpl Size, for: Tuple do
      def size(tuple), do: tuple_size(tuple)
    end

Finally, we can use the `Size` protocol to call the correct implementation:

    Size.size({1, 2})
    # => 2
    Size.size(%{key: :value})
    # => 1

Note that we didn't implement it for lists as we don't have the
`size` information on lists, rather its value needs to be
computed with `length`.

The data structure you are implementing the protocol for
must be the first argument to all functions defined in the
protocol.

It is possible to implement protocols for all Elixir types:

- Structs (see the "Protocols and Structs" section below)
- `Tuple`
- `Atom`
- `List`
- `BitString`
- `Integer`
- `Float`
- `Function`
- `PID`
- `Map`
- `Port`
- `Reference`
- `Any` (see the "Fallback to `Any`" section below)

## Protocols and Structs

The real benefit of protocols comes when mixed with structs.
For instance, Elixir ships with many data types implemented as
structs, like `MapSet`. We can implement the `Size` protocol
for those types as well:

    defimpl Size, for: MapSet do
      def size(map_set), do: MapSet.size(map_set)
    end

When implementing a protocol for a struct, the `:for` option can
be omitted if the `defimpl/3` call is inside the module that defines
the struct:

    defmodule User do
      defstruct [:email, :name]
    
      defimpl Size do
        # two fields
        def size(%User{}), do: 2
      end
    end

If a protocol implementation is not found for a given type,
invoking the protocol will raise unless it is configured to
fall back to `Any`. Conveniences for building implementations
on top of existing ones are also available, look at `defstruct/1`
for more information about deriving protocols.

## Fallback to `Any`

In some cases, it may be convenient to provide a default
implementation for all types. This can be achieved by setting
the `@fallback_to_any` attribute to `true` in the protocol
definition:

    defprotocol Size do
      @fallback_to_any true
      def size(data)
    end

The `Size` protocol can now be implemented for `Any`:

    defimpl Size, for: Any do
      def size(_), do: 0
    end

Although the implementation above is arguably not a reasonable
one. For example, it makes no sense to say a PID or an integer
have a size of `0`. That's one of the reasons why `@fallback_to_any`
is an opt-in behavior. For the majority of protocols, raising
an error when a protocol is not implemented is the proper behavior.

## Multiple implementations

Protocols can also be implemented for multiple types at once:

    defprotocol Reversible do
      def reverse(term)
    end
    
    defimpl Reversible, for: [Map, List] do
      def reverse(term), do: Enum.reverse(term)
    end

Inside `defimpl/3`, you can use `@protocol` to access the protocol
being implemented and `@for` to access the module it is being
defined for.

## Types

Defining a protocol automatically defines a zero-arity type named `t`, which
can be used as follows:

    @spec print_size(Size.t()) :: :ok
    def print_size(data) do
      result =
        case Size.size(data) do
          0 -> "data has no items"
          1 -> "data has one item"
          n -> "data has #{n} items"
        end
    
      IO.puts(result)
    end

The `@spec` above expresses that all types allowed to implement the
given protocol are valid argument types for the given function.

## Configuration

The following module attributes are available to configure a protocol:

- `@fallback_to_any` - when true, enables protocol dispatch to
  fallback to any

- `@undefined_impl_description` - a string with additional description
  to be used on `Protocol.UndefinedError` when looking up the implementation
  fails. This option is only applied if `@fallback_to_any` is not set to true

## Consolidation

In order to speed up protocol dispatching, whenever all protocol implementations
are known up-front, typically after all Elixir code in a project is compiled,
Elixir provides a feature called *protocol consolidation*. Consolidation directly
links protocols to their implementations in a way that invoking a function from a
consolidated protocol is equivalent to invoking two remote functions - one to
identify the correct implementation, and another to call the implementation.

Protocol consolidation is applied by default to all Mix projects during compilation.
This may be an issue during test. For instance, if you want to implement a protocol
during test, the implementation will have no effect, as the protocol has already been
consolidated. One possible solution is to include compilation directories that are
specific to your test environment in your mix.exs:

    def project do
      ...
      elixirc_paths: elixirc_paths(Mix.env())
      ...
    end
    
    defp elixirc_paths(:test), do: ["lib", "test/support"]
    defp elixirc_paths(_), do: ["lib"]

And then you can define the implementations specific to the test environment
inside `test/support/some_file.ex`.

Another approach is to disable protocol consolidation during tests in your
mix.exs:

    def project do
      ...
      consolidate_protocols: Mix.env() != :test
      ...
    end

If you are using `Mix.install/2`, you can do by passing the `consolidate_protocols`
option:

    Mix.install(
      deps,
      consolidate_protocols: false
    )

Although doing so is not recommended as it may affect the performance of
your code.

Finally, note all protocols are compiled with `debug_info` set to `true`,
regardless of the option set by the `elixirc` compiler. The debug info is
used for consolidation and it is removed after consolidation unless
globally set.

## Callbacks

### __deriving__(module, term)
*(optional)* 
```elixir
@macrocallback __deriving__(module(), term()) :: Macro.t()
```

An optional callback to be implemented by protocol authors for custom deriving.

It must return a quoted expression that implements the protocol for the given module.

See `Protocol.derive/3` for an example.

### __protocol__(atom)

```elixir
@callback __protocol__(:consolidated?) :: boolean()
@callback __protocol__(:functions) :: [{atom(), arity()}]
@callback __protocol__(:impls) :: {:consolidated, [module()]} | :not_consolidated
@callback __protocol__(:module) :: module()
```

A function available in all protocol definitions that returns protocol metadata.

### impl_for(term)

```elixir
@callback impl_for(term()) :: module() | nil
```

A function available in all protocol definitions that returns the implementation
for the given `term` or nil.

If `@fallback_to_any` is true, `nil` is never returned.

### impl_for!(term)

```elixir
@callback impl_for!(term()) :: module()
```

A function available in all protocol definitions that returns the implementation
for the given `term` or raises.

If `@fallback_to_any` is true, it never raises.

## Functions

### assert_impl!(protocol, base)

```elixir
@spec assert_impl!(module(), module()) :: :ok
```

Checks if the given module is loaded and is an implementation
of the given protocol.

Returns `:ok` if so, otherwise raises `ArgumentError`.

### assert_protocol!(module)

```elixir
@spec assert_protocol!(module()) :: :ok
```

Checks if the given module is loaded and is protocol.

Returns `:ok` if so, otherwise raises `ArgumentError`.

### consolidate(protocol, types)

```elixir
@spec consolidate(module(), [module()]) ::
  {:ok, binary()} | {:error, :not_a_protocol} | {:error, :no_beam_info}
```

Receives a protocol and a list of implementations and
consolidates the given protocol.

Consolidation happens by changing the protocol `impl_for`
in the abstract format to have fast lookup rules. Usually
the list of implementations to use during consolidation
are retrieved with the help of `extract_impls/2`.

It returns the updated version of the protocol bytecode.
If the first element of the tuple is `:ok`, it means
the protocol was consolidated.

A given bytecode or protocol implementation can be checked
to be consolidated or not by analyzing the protocol
attribute:

    Protocol.consolidated?(Enumerable)

This function does not load the protocol at any point
nor loads the new bytecode for the compiled module.
However, each implementation must be available and
it will be loaded.

### consolidated?(protocol)

```elixir
@spec consolidated?(module()) :: boolean()
```

Returns `true` if the protocol was consolidated.

### derive(protocol, module, options \\ [])
*(macro)* 


Derives the `protocol` for `module` with the given options.

Every time you derive a protocol, Elixir will verify if the protocol
has implemented the `c:Protocol.__deriving__/2` callback. If so,
the callback will be invoked and it should define the implementation
module. Otherwise an implementation that simply points to the `Any`
implementation is automatically derived.

#### Examples

    defprotocol Derivable do
      @impl true
      defmacro __deriving__(module, options) do
        # If you need to load struct metadata, you may call:
        # struct_info = Macro.struct_info!(module, __CALLER__)
    
        quote do
          defimpl Derivable, for: unquote(module) do
            def ok(arg) do
              {:ok, arg, unquote(options)}
            end
          end
        end
      end
    
      def ok(arg)
    end

Once the protocol is defined, there are two ways it can be
derived. The first is by using the `@derive` module attribute
by the time you define the struct:

    defmodule ImplStruct do
      @derive [Derivable]
      defstruct a: 0, b: 0
    end
    
    Derivable.ok(%ImplStruct{})
    #=> {:ok, %ImplStruct{a: 0, b: 0}, []}

If the struct has already been defined, you can call this macro:

    require Protocol
    Protocol.derive(Derivable, ImplStruct, :oops)
    Derivable.ok(%ImplStruct{a: 1, b: 1})
    #=> {:ok, %ImplStruct{a: 1, b: 1}, :oops}

### extract_impls(protocol, paths)

```elixir
@spec extract_impls(module(), [charlist() | String.t()]) :: [atom()]
```

Extracts all types implemented for the given protocol from
the given paths.

The paths can be either a charlist or a string. Internally
they are worked on as charlists, so passing them as lists
avoid extra conversion.

Does not load any of the implementations.

#### Examples

    # Get Elixir's ebin directory path and retrieve all protocols
    iex> path = Application.app_dir(:elixir, "ebin")
    iex> mods = Protocol.extract_impls(Enumerable, [path])
    iex> List in mods
    true

### extract_protocols(paths)

```elixir
@spec extract_protocols([charlist() | String.t()]) :: [atom()]
```

Extracts all protocols from the given paths.

The paths can be either a charlist or a string. Internally
they are worked on as charlists, so passing them as lists
avoid extra conversion.

Does not load any of the protocols.

#### Examples

    # Get Elixir's ebin directory path and retrieve all protocols
    iex> path = Application.app_dir(:elixir, "ebin")
    iex> mods = Protocol.extract_protocols([path])
    iex> Enumerable in mods
    true



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
