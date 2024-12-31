# Module behaviour
(Elixir v1.18.0-dev)

Provides functions to deal with modules during compilation time.

It allows a developer to dynamically add, delete and register
attributes, attach documentation and so forth.

After a module is compiled, using many of the functions in
this module will raise errors, since it is out of their scope
to inspect runtime data. Most of the runtime data can be inspected
via the [`__info__/1`](\`c:Module.__info__/1\`) function attached to
each compiled module.

## Module attributes

Each module can be decorated with one or more attributes. The following ones
are currently defined by Elixir:

### `@after_compile`

A hook that will be invoked right after the current module is compiled.
Accepts a module or a `{module, function_name}`. See the "Compile callbacks"
section below.

### `@after_verify` (since v1.14.0)

A hook that will be invoked right after the current module is verified for
undefined functions, deprecations, etc. Accepts a module or a `{module, function_name}`.
See the "Compile callbacks" section below.

### `@before_compile`

A hook that will be invoked before the module is compiled.
Accepts a module or a `{module, function_or_macro_name}` tuple.
See the "Compile callbacks" section below.

### `@behaviour`

Note the British spelling\!

Behaviours can be referenced by modules to ensure they implement
required specific function signatures defined by `@callback`.

For example, you could specify a `URI.Parser` behaviour as follows:

    defmodule URI.Parser do
      @doc "Defines a default port"
      @callback default_port() :: integer
    
      @doc "Parses the given URL"
      @callback parse(uri_info :: URI.t()) :: URI.t()
    end

And then a module may use it as:

    defmodule URI.HTTP do
      @behaviour URI.Parser
      def default_port(), do: 80
      def parse(info), do: info
    end

If the behaviour changes or `URI.HTTP` does not implement
one of the callbacks, a warning will be raised.

For detailed documentation, see the
[behaviour typespec documentation](typespecs.md#behaviours).

### `@impl` (since v1.5.0)

To aid in the correct implementation of behaviours, you may optionally declare
`@impl` for implemented callbacks of a behaviour. This makes callbacks
explicit and can help you to catch errors in your code. The compiler will warn
in these cases:

- if you mark a function with `@impl` when that function is not a callback.

- if you don't mark a function with `@impl` when other functions are marked
  with `@impl`. If you mark one function with `@impl`, you must mark all
  other callbacks for that behaviour as `@impl`.

`@impl` works on a per-context basis. If you generate a function through a macro
and mark it with `@impl`, that won't affect the module where that function is
generated in.

`@impl` also helps with maintainability by making it clear to other developers
that the function is implementing a callback.

Using `@impl`, the example above can be rewritten as:

    defmodule URI.HTTP do
      @behaviour URI.Parser
    
      @impl true
      def default_port(), do: 80
    
      @impl true
      def parse(info), do: info
    end

You may pass either `false`, `true`, or a specific behaviour to `@impl`.

    defmodule Foo do
      @behaviour Bar
      @behaviour Baz
    
      # Will warn if neither Bar nor Baz specify a callback named bar/0.
      @impl true
      def bar(), do: :ok
    
      # Will warn if Baz does not specify a callback named baz/0.
      @impl Baz
      def baz(), do: :ok
    end

The code is now more readable, as it is now clear which functions are
part of your API and which ones are callback implementations. To reinforce this
idea, `@impl true` automatically marks the function as `@doc false`, disabling
documentation unless `@doc` is explicitly set.

### `@compile`

Defines options for module compilation. This is used to configure
both Elixir and Erlang compilers, as any other compilation pass
added by external tools. For example:

    defmodule MyModule do
      @compile {:inline, my_fun: 1}
    
      def my_fun(arg) do
        to_string(arg)
      end
    end

Multiple uses of `@compile` will accumulate instead of overriding
previous ones. See the "Compile options" section below.

### `@deprecated` (since v1.6.0)

Provides the deprecation reason for a function. For example:

    defmodule Keyword do
      @deprecated "Use Kernel.length/1 instead"
      def size(keyword) do
        length(keyword)
      end
    end

The Mix compiler automatically looks for calls to deprecated modules
and emit warnings during compilation.

Using the `@deprecated` attribute will also be reflected in the
documentation of the given function and macro. You can choose between
the `@deprecated` attribute and the documentation metadata to provide
hard-deprecations (with warnings) and soft-deprecations (without warnings):

This is a soft-deprecation as it simply annotates the documentation
as deprecated:

    @doc deprecated: "Use Kernel.length/1 instead"
    def size(keyword)

This is a hard-deprecation as it emits warnings and annotates the
documentation as deprecated:

    @deprecated "Use Kernel.length/1 instead"
    def size(keyword)

Currently `@deprecated` only supports functions and macros. However
you can use the `:deprecated` key in the annotation metadata to
annotate the docs of modules, types and callbacks too.

We recommend using this feature with care, especially library authors.
Deprecating code always pushes the burden towards library users. We
also recommend for deprecated functionality to be maintained for long
periods of time, even after deprecation, giving developers plenty of
time to update (except for cases where keeping the deprecated API is
undesired, such as in the presence of security issues).

### `@doc` and `@typedoc`

Provides documentation for the entity that follows the attribute.
`@doc` is to be used with a function, macro, callback, or
macrocallback, while `@typedoc` with a type (public or opaque).

Accepts one of these:

- a string (often a heredoc)
- `false`, which will make the entity invisible to documentation-extraction
  tools like [`ExDoc`](https://hexdocs.pm/ex_doc/)
- a keyword list, since Elixir 1.7.0

For example:

    defmodule MyModule do
      @typedoc "This type"
      @typedoc since: "1.1.0"
      @type t :: term
    
      @doc "Hello world"
      @doc since: "1.1.0"
      def hello do
        "world"
      end
    
      @doc """
      Sums `a` to `b`.
      """
      def sum(a, b) do
        a + b
      end
    end

As can be seen in the example above, since Elixir 1.7.0 `@doc` and `@typedoc`
also accept a keyword list that serves as a way to provide arbitrary metadata
about the entity. Tools like [`ExDoc`](https://hexdocs.pm/ex_doc/) and
`IEx` may use this information to display annotations. A common use
case is the `:since` key, which may be used to annotate in which version the
function was introduced.

As illustrated in the example, it is possible to use these attributes
more than once before an entity. However, the compiler will warn if
used twice with binaries as that replaces the documentation text from
the preceding use. Multiple uses with keyword lists will merge the
lists into one.

Note that since the compiler also defines some additional metadata,
there are a few reserved keys that will be ignored and warned if used.
Currently these are: `:opaque` and `:defaults`.

Once this module is compiled, this information becomes available via
the `Code.fetch_docs/1` function.

### `@dialyzer`

Defines warnings to request or suppress when using `:dialyzer`.

Accepts an atom, a tuple, or a list of atoms and tuples. For example:

    defmodule MyModule do
      @dialyzer {:nowarn_function, [my_fun: 1]}
    
      def my_fun(arg) do
        M.not_a_function(arg)
      end
    end

For the list of supported warnings, see [`:dialyzer` module](\`:dialyzer\`).

Multiple uses of `@dialyzer` will accumulate instead of overriding
previous ones.

### `@external_resource`

Specifies an external resource for the current module.

Sometimes a module embeds information from an external file. This
attribute allows the module to annotate which external resources
have been used.

Tools may use this information to ensure the module is recompiled
in case any of the external resources change, see for example:
[`mix compile.elixir`](https://hexdocs.pm/mix/Mix.Tasks.Compile.Elixir.html).

The specified file path provided is interpreted as relative to
the folder containing the project's `mix.exs`, which is the
current working directory, not the file where `@external_resource`
is declared.

If the external resource does not exist, the module still has
a dependency on it, causing the module to be recompiled as soon
as the file is added.

For more control over when a module is recompiled, see
[`__mix_recompile__?/0`](\`m:Mix.Tasks.Compile.Elixir#module-__mix_recompile__-0\`).

### `@file`

Changes the filename used in stacktraces for the function or macro that
follows the attribute, such as:

    defmodule MyModule do
      @doc "Hello world"
      @file "hello.ex"
      def hello do
        "world"
      end
    end

Note that this is only valid for exceptions/diagnostics that come from the
definition inner scope (which includes its patterns and guards). For example:

    defmodule MyModule do # <---- module definition
      @file "hello.ex"
      defp unused(a) do # <---- function definition
        "world" # <---- function scope
      end
    
      @file "bye.ex"
      def unused(_), do: true
    end

If you run this code with the second "unused" definition commented, you will
see that `hello.ex` is used as the stacktrace when reporting warnings, but if
you uncomment it you'll see that the error will not mention `bye.ex`, because
it's a module-level error rather than an expression-level error.

### `@moduledoc`

Provides documentation for the current module.

    defmodule MyModule do
      @moduledoc """
      A very useful module.
      """
      @moduledoc authors: ["Alice", "Bob"]
    end

Accepts a string (often a heredoc) or `false` where `@moduledoc false`
will make the module invisible to documentation extraction tools like
[`ExDoc`](https://hexdocs.pm/ex_doc/).

Similarly to `@doc` also accepts a keyword list to provide metadata
about the module. For more details, see the documentation of `@doc`
above.

Once this module is compiled, this information becomes available via
the `Code.fetch_docs/1` function.

### `@nifs` (since v1.16.0)

A list of functions and their arities which will be overridden
by a native implementation (NIF).

    defmodule MyLibrary.MyModule do
      @nifs [foo: 1, bar: 2]
    
      def foo(arg1), do: :erlang.nif_error(:not_loaded)
      def bar(arg1, arg2), do: :erlang.nif_error(:not_loaded)
    end

See the Erlang documentation for more information:
https://www.erlang.org/doc/man/erl\_nif

### `@on_definition`

A hook that will be invoked when each function or macro in the current
module is defined. Useful when annotating functions.

Accepts a module or a `{module, function_name}` tuple. The function
must take 6 arguments:

- the module environment
- the kind of the function/macro: `:def`, `:defp`, `:defmacro`, or `:defmacrop`
- the function/macro name
- the list of quoted arguments
- the list of quoted guards
- the quoted function body

If the function/macro being defined has multiple clauses, the hook will
be called for each clause.

Unlike other hooks, `@on_definition` will only invoke functions and
never macros. This is to avoid `@on_definition` callbacks from
redefining functions that have just been defined in favor of more
explicit approaches.

When just a module is provided, the function is assumed to be
`__on_definition__/6`.

#### Example

    defmodule Hooks do
      def on_def(_env, kind, name, args, guards, body) do
        IO.puts("Defining #{kind} named #{name} with args:")
        IO.inspect(args)
        IO.puts("and guards")
        IO.inspect(guards)
        IO.puts("and body")
        IO.puts(Macro.to_string(body))
      end
    end
    
    defmodule MyModule do
      @on_definition {Hooks, :on_def}
    
      def hello(arg) when is_binary(arg) or is_list(arg) do
        "Hello" <> to_string(arg)
      end
    
      def hello(_) do
        :ok
      end
    end

### `@on_load`

A hook that will be invoked whenever the module is loaded.

Accepts the function name (as an atom) of a function in the current module.
The function must have an arity of 0 (no arguments). If the function does
not return `:ok`, the loading of the module will be aborted.
For example:

    defmodule MyModule do
      @on_load :load_check
    
      def load_check do
        if some_condition() do
          :ok
        else
          :abort
        end
      end
    
      def some_condition do
        false
      end
    end

### `@vsn`

Specify the module version. Accepts any valid Elixir value, for example:

    defmodule MyModule do
      @vsn "1.0"
    end

### Struct attributes

- `@derive` - derives an implementation for the given protocol for the
  struct defined in the current module

- `@enforce_keys` - ensures the given keys are always set when building
  the struct defined in the current module

See `defstruct/1` for more information on building and using structs.

### Typespec attributes

The following attributes are part of typespecs and are also built-in in
Elixir:

- `@type` - defines a type to be used in `@spec`
- `@typep` - defines a private type to be used in `@spec`
- `@opaque` - defines an opaque type to be used in `@spec`
- `@spec` - provides a specification for a function
- `@callback` - provides a specification for a behaviour callback (and generates
  a `behaviour_info/1` function in the module, see below)
- `@macrocallback` - provides a specification for a macro behaviour callback
- `@optional_callbacks` - specifies which behaviour callbacks and macro
  behaviour callbacks are optional
- `@impl` - declares an implementation of a callback function or macro

For detailed documentation, see the [typespec documentation](typespecs.md).

### Custom attributes

In addition to the built-in attributes outlined above, custom attributes may
also be added. Custom attributes are expressed using the `@/1` operator followed
by a valid variable name. The value given to the custom attribute must be a valid
Elixir value:

    defmodule MyModule do
      @custom_attr [some: "stuff"]
    end

For more advanced options available when defining custom attributes, see
`register_attribute/3`.

## Compile callbacks

There are three compilation callbacks, invoked in this order:
`@before_compile`, `@after_compile`, and `@after_verify`.
They are described next.

### `@before_compile`

A hook that will be invoked before the module is compiled. This is
often used to change how the current module is being compiled.

Accepts a module or a `{module, function_or_macro_name}` tuple. The
function/macro must take one argument: the module environment. If
it's a macro, its returned value will be injected at the end of the
module definition before the compilation starts.

When just a module is provided, the function/macro is assumed to be
`__before_compile__/1`.

Callbacks will run in the order they are registered. Any overridable
definition will be made concrete before the first callback runs.
A definition may be made overridable again in another before compile
callback and it will be made concrete one last time after all callbacks
run.

*Note*: the callback function/macro must be placed in a separate module
(because when the callback is invoked, the current module does not yet exist).

#### Example

    defmodule A do
      defmacro __before_compile__(_env) do
        quote do
          def hello, do: "world"
        end
      end
    end
    
    defmodule B do
      @before_compile A
    end
    
    B.hello()
    #=> "world"

### `@after_compile`

A hook that will be invoked right after the current module is compiled.

Accepts a module or a `{module, function_name}` tuple. The function
must take two arguments: the module environment and its bytecode.
When just a module is provided, the function is assumed to be
`__after_compile__/2`.

Callbacks will run in the order they are registered.

`Module` functions expecting not yet compiled modules (such as `definitions_in/1`)
are still available at the time `@after_compile` is invoked.

#### Example

    defmodule MyModule do
      @after_compile __MODULE__
    
      def __after_compile__(env, _bytecode) do
        IO.inspect(env)
      end
    end

### `@after_verify`

A hook that will be invoked right after the current module is verified for
undefined functions, deprecations, etc. A module is always verified after
it is compiled. In Mix projects, a module is also verified when any of its
runtime dependencies change. Therefore this is useful to perform verification
of the current module while avoiding compile-time dependencies. Given the
callback is invoked under different scenarios, Elixir provides no guarantees
of when in the compilation cycle nor in which process the callback runs.

Accepts a module or a `{module, function_name}` tuple. The function
must take one argument: the module name. When just a module is provided,
the function is assumed to be `__after_verify__/1`.

Callbacks will run in the order they are registered.

`Module` functions expecting not yet compiled modules are no longer available
at the time `@after_verify` is invoked.

#### Example

    defmodule MyModule do
      @after_verify __MODULE__
    
      def __after_verify__(module) do
        IO.inspect(module)
        :ok
      end
    end

## Compile options

The `@compile` attribute accepts different options that are used by both
Elixir and Erlang compilers. Some of the common use cases are documented
below:

- `@compile :debug_info` - includes `:debug_info` regardless of the
  corresponding setting in `Code.get_compiler_option/1`

- `@compile {:debug_info, false}` - disables `:debug_info` regardless
  of the corresponding setting in `Code.get_compiler_option/1`. Note
  disabling `:debug_info` is not recommended as it removes the ability
  of the Elixir compiler and other tools to static analyse the code.
  If you want to remove the `:debug_info` while deploying, tools like
  `mix release` already do such by default.

- `@compile {:inline, some_fun: 2, other_fun: 3}` - inlines the given
  name/arity pairs. Inlining is applied locally, calls from another
  module are not affected by this option

- `@compile {:autoload, false}` - disables automatic loading of
  modules after compilation. Instead, the module will be loaded after
  it is dispatched to

- `@compile {:no_warn_undefined, Mod}` or
  `@compile {:no_warn_undefined, {Mod, fun, arity}}` - does not warn if
  the given module or the given `Mod.fun/arity` are not defined

## Generated functions

Sometimes the compiler will generate public functions within modules. These
are documented below.

### `behaviour_info/1`

This function is generated for modules that define a behaviour, that is,
that have one or more `@callback` definitions. The signature for this function,
expressed as a spec, is:

    @spec behaviour_info(:callbacks) :: [function_info]
      when function_info: {function_name :: atom(), arity :: non_neg_integer()}
    
    @spec behaviour_info(:optional_callbacks) :: [function_info]
      when function_info: {function_name :: atom(), arity :: non_neg_integer()}

`behaviour_info(:callbacks)` includes optional callbacks.

For example:

    iex> Enum.sort(GenServer.behaviour_info(:callbacks))
    [
      code_change: 3,
      format_status: 1,
      format_status: 2,
      handle_call: 3,
      handle_cast: 2,
      handle_continue: 2,
      handle_info: 2,
      init: 1,
      terminate: 2
    ]

### `module_info/0`

This function is generated for all modules. It returns all the attributes
returned by `module_info/1` (see below), but as a single keyword list. See also the
[Erlang documentation](https://www.erlang.org/doc/system/modules.html#module_info-0).

### `module_info/1`

This function is generated for all modules and returns
information about the module. The signature for this function,
expressed as a spec, is:

    @spec module_info(:module) :: module() # Returns the module itself
    @spec module_info(:attributes) :: keyword()
    @spec module_info(:compile) :: keyword()
    @spec module_info(:md5) :: binary()
    @spec module_info(:nifs) :: module()
    @spec module_info(:exports) :: [function_info]
      when function_info: {function_name :: atom(), arity :: non_neg_integer()}
    @spec module_info(:functions) :: [function_info]
      when function_info: {function_name :: atom(), arity :: non_neg_integer()}

For example:

    iex> URI.module_info(:module)
    URI
    iex> {:decode_www_form, 1} in URI.module_info(:exports)
    true

For more information about `module_info/1`, also check out the [Erlang
documentation](https://www.erlang.org/doc/system/modules.html#module_info-1).

### `__info__/1`

This function is generated for all modules. It's similar to `module_info/1` but
includes some additional Elixir-specific information, such as struct and macro
information. For documentation, see `c:Module.__info__/1`.

## Types

### def_kind()

```elixir
@type def_kind() :: :def | :defp | :defmacro | :defmacrop
```



### definition()

```elixir
@type definition() :: {atom(), arity()}
```



## Callbacks

### __info__(atom)

```elixir
@callback __info__(:attributes) :: keyword()
@callback __info__(:compile) :: [term()]
@callback __info__(:functions) :: keyword()
@callback __info__(:macros) :: keyword()
@callback __info__(:md5) :: binary()
@callback __info__(:module) :: module()
@callback __info__(:struct) :: [%{:field =&gt; atom(), optional(:default) =&gt; term()}] | nil
```

Provides runtime information about functions, macros, and other information
defined by the module.

Each module gets an `__info__/1` function when it's compiled. The function
takes one of the following items:

- `:attributes` - a keyword list with all persisted attributes

- `:compile` - a list with compiler metadata

- `:functions` - a keyword list of public functions and their arities

- `:macros` - a keyword list of public macros and their arities

- `:md5` - the MD5 of the module

- `:module` - the module atom name

- `:struct` - (since v1.14.0) if the module defines a struct and if so each field in order

## Functions

### attributes_in(module)
*(since 1.13.0)* 
```elixir
@spec attributes_in(module()) :: [atom()]
```

Returns all module attributes names defined in `module`.

This function can only be used on modules that have not yet been compiled.

#### Examples

    defmodule Example do
      @foo 1
      Module.register_attribute(__MODULE__, :bar, accumulate: true)
    
      :foo in Module.attributes_in(__MODULE__)
      #=> true
    
      :bar in Module.attributes_in(__MODULE__)
      #=> true
    end

### concat(list)

```elixir
@spec concat([binary() | atom()]) :: atom()
```

Concatenates a list of aliases and returns a new alias.

It handles binaries and atoms.

#### Examples

    iex> Module.concat([Foo, Bar])
    Foo.Bar
    
    iex> Module.concat([Foo, "Bar"])
    Foo.Bar

### concat(left, right)

```elixir
@spec concat(binary() | atom(), binary() | atom()) :: atom()
```

Concatenates two aliases and returns a new alias.

It handles binaries and atoms.

#### Examples

    iex> Module.concat(Foo, Bar)
    Foo.Bar
    
    iex> Module.concat(Foo, "Bar")
    Foo.Bar

### create(module, quoted, opts)

```elixir
@spec create(module(), Macro.t(), Macro.Env.t() | keyword()) ::
  {:module, module(), binary(), term()}
```

Creates a module with the given name and defined by
the given quoted expressions.

The line where the module is defined and its file **must**
be passed as options. See `Code.env_for_eval/1` for a complete
list of options.

It returns a tuple of shape `{:module, module, binary, term}`
where `module` is the module name, `binary` is the module
bytecode and `term` is the result of the last expression in
`quoted`.

Similar to `Kernel.defmodule/2`, the binary will only be
written to disk as a `.beam` file if `Module.create/3` is
invoked in a file that is currently being compiled.

#### Examples

    contents =
      quote do
        def world, do: true
      end
    
    Module.create(Hello, contents, Macro.Env.location(__ENV__))
    
    Hello.world()
    #=> true

#### Differences from `defmodule`

`Module.create/3` works similarly to `Kernel.defmodule/2`
and return the same results. While one could also use
`Kernel.defmodule/2` to define modules dynamically, this function
is preferred when the module body is given by a quoted
expression.

Another important distinction is that `Module.create/3`
allows you to control the environment variables used
when defining the module, while `Kernel.defmodule/2`
automatically uses the environment it is invoked at.

### defines?(module, tuple)

```elixir
@spec defines?(module(), definition()) :: boolean()
```

Checks if the module defines the given function or macro.

Use `defines?/3` to assert for a specific type.

This function can only be used on modules that have not yet been compiled.
Use `Kernel.function_exported?/3` and `Kernel.macro_exported?/3` to check for
public functions and macros respectively in compiled modules.

Note that `defines?` returns `false` for functions and macros that have
been defined but then marked as overridable and no other implementation
has been provided. You can check the overridable status by calling
`overridable?/2`.

#### Examples

    defmodule Example do
      Module.defines?(__MODULE__, {:version, 0}) #=> false
      def version, do: 1
      Module.defines?(__MODULE__, {:version, 0}) #=> true
    end

### defines?(module, tuple, def_kind)

```elixir
@spec defines?(module(), definition(), def_kind()) :: boolean()
```

Checks if the module defines a function or macro of the
given `kind`.

`kind` can be any of `:def`, `:defp`, `:defmacro`, or `:defmacrop`.

This function can only be used on modules that have not yet been compiled.
Use `Kernel.function_exported?/3` and `Kernel.macro_exported?/3` to check for
public functions and macros respectively in compiled modules.

#### Examples

    defmodule Example do
      Module.defines?(__MODULE__, {:version, 0}, :def) #=> false
      def version, do: 1
      Module.defines?(__MODULE__, {:version, 0}, :def) #=> true
    end

### defines_type?(module, definition)
*(since 1.7.0)* 
```elixir
@spec defines_type?(module(), definition()) :: boolean()
```

Checks if the current module defines the given type (private, opaque or not).

This function is only available for modules being compiled.

### definitions_in(module)

```elixir
@spec definitions_in(module()) :: [definition()]
```

Returns all functions and macros defined in `module`.

It returns a list with all defined functions and macros, public and private,
in the shape of `[{name, arity}, ...]`.

This function can only be used on modules that have not yet been compiled.
Use the `c:Module.__info__/1` callback to get the public functions and macros in
compiled modules.

#### Examples

    defmodule Example do
      def version, do: 1
      defmacrop test(arg), do: arg
      Module.definitions_in(__MODULE__) #=> [{:version, 0}, {:test, 1}]
    end

### definitions_in(module, kind)

```elixir
@spec definitions_in(module(), def_kind()) :: [definition()]
```

Returns all functions defined in `module`, according
to its kind.

This function can only be used on modules that have not yet been compiled.
Use the `c:Module.__info__/1` callback to get the public functions and macros in
compiled modules.

#### Examples

    defmodule Example do
      def version, do: 1
      Module.definitions_in(__MODULE__, :def)  #=> [{:version, 0}]
      Module.definitions_in(__MODULE__, :defp) #=> []
    end

### delete_attribute(module, key)

```elixir
@spec delete_attribute(module(), atom()) :: term()
```

Deletes the entry (or entries) for the given module attribute.

It returns the deleted attribute value. If the attribute has not
been set nor configured to accumulate, it returns `nil`.

If the attribute is set to accumulate, then this function always
returns a list. Deleting the attribute removes existing entries
but the attribute will still accumulate.

#### Examples

    defmodule MyModule do
      Module.put_attribute(__MODULE__, :custom_threshold_for_lib, 10)
      Module.delete_attribute(__MODULE__, :custom_threshold_for_lib)
    end

### delete_definition(module, arg)
*(since 1.12.0)* 
```elixir
@spec delete_definition(module(), definition()) :: boolean()
```

Deletes a definition from a module.

It returns `true` if the definition exists and it was removed,
otherwise it returns `false`.

### eval_quoted(module_or_env, quoted, binding \\ [], opts \\ [])


This function is deprecated. Use Code.eval_quoted/3 instead.


### get_attribute(module, key, default \\ nil)

```elixir
@spec get_attribute(module(), atom(), term()) :: term()
```

Gets the given attribute from a module.

If the attribute was marked with `accumulate` with
`Module.register_attribute/3`, a list is always returned.
`nil` is returned if the attribute has not been marked with
`accumulate` and has not been set to any value.

The `@` macro compiles to a call to this function. For example,
the following code:

    @foo

Expands to something akin to:

    Module.get_attribute(__MODULE__, :foo)

This function can only be used on modules that have not yet been compiled.
Use the `c:Module.__info__/1` callback to get all persisted attributes, or
`Code.fetch_docs/1` to retrieve all documentation related attributes in
compiled modules.

#### Examples

    defmodule Foo do
      Module.put_attribute(__MODULE__, :value, 1)
      Module.get_attribute(__MODULE__, :value) #=> 1
    
      Module.get_attribute(__MODULE__, :value, :default) #=> 1
      Module.get_attribute(__MODULE__, :not_found, :default) #=> :default
    
      Module.register_attribute(__MODULE__, :value, accumulate: true)
      Module.put_attribute(__MODULE__, :value, 1)
      Module.get_attribute(__MODULE__, :value) #=> [1]
    end

### get_definition(module, arg, options \\ [])
*(since 1.12.0)* 
```elixir
@spec get_definition(module(), definition(), keyword()) ::
  {:v1, def_kind(), meta :: keyword(),
   [
     {meta :: keyword(), arguments :: [Macro.t()], guards :: [Macro.t()],
      Macro.t()}
   ]}
  | nil
```

Returns the definition for the given name-arity pair.

It returns a tuple with the `version`, the `kind`,
the definition `metadata`, and a list with each clause.
Each clause is a four-element tuple with metadata,
the arguments, the guards, and the clause AST.

The clauses are returned in the Elixir AST but a subset
that has already been expanded and normalized. This makes
it useful for analyzing code but it cannot be reinjected
into the module as it will have lost some of its original
context. Given this AST representation is mostly internal,
it is versioned and it may change at any time. Therefore,
**use this API with caution**.

#### Options

- `:skip_clauses` (since v1.14.0) - returns `[]` instead
  of returning the clauses. This is useful when there is
  only an interest in fetching the kind and the metadata

### get_last_attribute(module, key, default \\ nil)
*(since 1.15.0)* 
```elixir
@spec get_last_attribute(module(), atom(), term()) :: term()
```

Gets the last set value of a given attribute from a module.

If the attribute was marked with `accumulate` with
`Module.register_attribute/3`, the previous value to have been set will be
returned. If the attribute does not accumulate, this call is the same as
calling `Module.get_attribute/3`.

This function can only be used on modules that have not yet been compiled.
Use the `c:Module.__info__/1` callback to get all persisted attributes, or
`Code.fetch_docs/1` to retrieve all documentation related attributes in
compiled modules.

#### Examples

    defmodule Foo do
      Module.put_attribute(__MODULE__, :value, 1)
      Module.get_last_attribute(__MODULE__, :value) #=> 1
    
      Module.get_last_attribute(__MODULE__, :not_found, :default) #=> :default
    
      Module.register_attribute(__MODULE__, :acc, accumulate: true)
      Module.put_attribute(__MODULE__, :acc, 1)
      Module.get_last_attribute(__MODULE__, :acc) #=> 1
      Module.put_attribute(__MODULE__, :acc, 2)
      Module.get_last_attribute(__MODULE__, :acc) #=> 2
    end

### has_attribute?(module, key)
*(since 1.10.0)* 
```elixir
@spec has_attribute?(module(), atom()) :: boolean()
```

Checks if the given attribute has been defined.

An attribute is defined if it has been registered with `register_attribute/3`
or assigned a value. If an attribute has been deleted with `delete_attribute/2`
it is no longer considered defined.

This function can only be used on modules that have not yet been compiled.

#### Examples

    defmodule MyModule do
      @value 1
      Module.register_attribute(__MODULE__, :other_value)
      Module.put_attribute(__MODULE__, :another_value, 1)
    
      Module.has_attribute?(__MODULE__, :value) #=> true
      Module.has_attribute?(__MODULE__, :other_value) #=> true
      Module.has_attribute?(__MODULE__, :another_value) #=> true
    
      Module.has_attribute?(__MODULE__, :undefined) #=> false
    
      Module.delete_attribute(__MODULE__, :value)
      Module.has_attribute?(__MODULE__, :value) #=> false
    end

### make_overridable(module, tuples)

```elixir
@spec make_overridable(module(), [definition()]) :: :ok
@spec make_overridable(module(), module()) :: :ok
```

Makes the given functions in `module` overridable.

An overridable function is lazily defined, allowing a
developer to customize it. See `Kernel.defoverridable/1` for
more information and documentation.

Once a function or a macro is marked as overridable, it will
no longer be listed under `definitions_in/1` or return true
when given to `defines?/2` until another implementation is
given.

### open?(module)

```elixir
@spec open?(module()) :: boolean()
```

Checks if a module is open.

A module is "open" if it is currently being defined and its attributes and
functions can be modified.

### overridable?(module, tuple)

```elixir
@spec overridable?(module(), definition()) :: boolean()
```

Returns `true` if `tuple` in `module` was marked as overridable
at some point.

Note `overridable?/2` returns `true` even if the definition was
already overridden. You can use `defines?/2` to see if a definition
exists or one is pending.

### overridables_in(module)
*(since 1.13.0)* 
```elixir
@spec overridables_in(module()) :: [atom()]
```

Returns all overridable definitions in `module`.

Note a definition is included even if it was was already overridden.
You can use `defines?/2` to see if a definition exists or one is pending.

This function can only be used on modules that have not yet been compiled.

#### Examples

    defmodule Example do
      def foo, do: 1
      def bar, do: 2
    
      defoverridable foo: 0, bar: 0
      def foo, do: 3
    
      [bar: 0, foo: 0] = Module.overridables_in(__MODULE__) |> Enum.sort()
    end

### put_attribute(module, key, value)

```elixir
@spec put_attribute(module(), atom(), term()) :: :ok
```

Puts a module attribute with `key` and `value` in the given `module`.

#### Examples

    defmodule MyModule do
      Module.put_attribute(__MODULE__, :custom_threshold_for_lib, 10)
    end

### register_attribute(module, attribute, options)

```elixir
@spec register_attribute(module(), atom(), accumulate: boolean(), persist: boolean()) ::
  :ok
```

Registers an attribute.

By registering an attribute, a developer is able to customize
how Elixir will store and accumulate the attribute values.

#### Options

When registering an attribute, two options can be given:

- `:accumulate` - several calls to the same attribute will
  accumulate instead of overriding the previous one. New attributes
  are always added to the top of the accumulated list.

- `:persist` - the attribute will be persisted in the Erlang
  Abstract Format. Useful when interfacing with Erlang libraries.

By default, both options are `false`. Once an attribute has been
set to accumulate or persist, the behaviour cannot be reverted.

#### Examples

    defmodule MyModule do
      Module.register_attribute(__MODULE__, :custom_threshold_for_lib, accumulate: true)
    
      @custom_threshold_for_lib 10
      @custom_threshold_for_lib 20
      @custom_threshold_for_lib #=> [20, 10]
    end

### reserved_attributes()
*(since 1.12.0)* 
```elixir
@spec reserved_attributes() :: map()
```

Returns information about module attributes used by Elixir.

See the "Module attributes" section in the module documentation for more
information on each attribute.

#### Examples

    iex> map = Module.reserved_attributes()
    iex> Map.has_key?(map, :moduledoc)
    true
    iex> Map.has_key?(map, :doc)
    true

### safe_concat(list)

```elixir
@spec safe_concat([binary() | atom()]) :: atom()
```

Concatenates a list of aliases and returns a new alias only if the alias
was already referenced.

If the alias was not referenced yet, fails with `ArgumentError`.
It handles binaries and atoms.

#### Examples

    iex> Module.safe_concat([List, Chars])
    List.Chars

### safe_concat(left, right)

```elixir
@spec safe_concat(binary() | atom(), binary() | atom()) :: atom()
```

Concatenates two aliases and returns a new alias only if the alias was
already referenced.

If the alias was not referenced yet, fails with `ArgumentError`.
It handles binaries and atoms.

#### Examples

    iex> Module.safe_concat(List, Chars)
    List.Chars

### spec_to_callback(module, definition)
*(since 1.7.0)* 
```elixir
@spec spec_to_callback(module(), definition()) :: boolean()
```

Copies the given spec as a callback.

Returns `true` if there is such a spec and it was copied as a callback.
If the function associated to the spec has documentation defined prior to
invoking this function, the docs are copied too.

### split(module)

```elixir
@spec split(module() | String.t()) :: [String.t(), ...]
```

Splits the given module name into binary parts.

`module` has to be an Elixir module, as `split/1` won't work with Erlang-style
modules (for example, `split(:lists)` raises an error).

`split/1` also supports splitting the string representation of Elixir modules
(that is, the result of calling `Atom.to_string/1` with the module name).

#### Examples

    iex> Module.split(Very.Long.Module.Name.And.Even.Longer)
    ["Very", "Long", "Module", "Name", "And", "Even", "Longer"]
    iex> Module.split("Elixir.String.Chars")
    ["String", "Chars"]



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
