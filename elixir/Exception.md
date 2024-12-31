# Exception behaviour
(Elixir v1.18.0-dev)

Functions for dealing with throw/catch/exit and exceptions.

This module also defines the behaviour required by custom
exceptions. To define your own, see `defexception/1`.

## Formatting functions

Several functions in this module help format exceptions.
Some of these functions expect the stacktrace as argument.
The stacktrace is typically available inside catch and
rescue by using the `__STACKTRACE__/0` variable.

Do not rely on the particular format returned by the
functions in this module. They may be changed in future releases
in order to better suit Elixir's tool chain. In other words,
by using the functions in this module it is guaranteed you will
format exceptions as in the current Elixir version being used.


## Types

### arity_or_args()

```elixir
@type arity_or_args() :: non_neg_integer() | list()
```



### kind()

```elixir
@type kind() :: :error | non_error_kind()
```

The kind handled by formatting functions


### location()

```elixir
@type location() :: keyword()
```



### non_error_kind()

```elixir
@type non_error_kind() :: :exit | :throw | {:EXIT, pid()}
```



### stacktrace()

```elixir
@type stacktrace() :: [stacktrace_entry()]
```



### stacktrace_entry()

```elixir
@type stacktrace_entry() ::
  {module(), atom(), arity_or_args(), location()}
  | {(... -&gt; any()), arity_or_args(), location()}
```



### t()

```elixir
@type t() :: %{
  :__struct__ =&gt; module(),
  :__exception__ =&gt; true,
  optional(atom()) =&gt; any()
}
```

The exception type


## Callbacks

### blame(t, stacktrace)
*(optional)* 
```elixir
@callback blame(t(), stacktrace()) :: {t(), stacktrace()}
```

Called from `Exception.blame/3` to augment the exception struct.

Can be used to collect additional information about the exception
or do some additional expensive computation.


### exception(term)

```elixir
@callback exception(term()) :: t()
```

Receives the arguments given to `raise/2` and returns the exception struct.

The default implementation accepts either a set of keyword arguments
that is merged into the struct or a string to be used as the exception's message.


### message(t)

```elixir
@callback message(t()) :: String.t()
```

Receives the exception struct and must return its message.

Many exceptions have a message field which by default is accessed
by this function. However, if an exception does not have a message field,
this function must be explicitly implemented.


## Functions

### blame(kind, error, stacktrace)
*(since 1.5.0)* 
```elixir
@spec blame(:error, any(), stacktrace()) :: {t(), stacktrace()}
@spec blame(non_error_kind(), payload, stacktrace()) :: {payload, stacktrace()}
when payload: var
```

Attaches information to exceptions for extra debugging.

This operation is potentially expensive, as it reads data
from the file system, parses beam files, evaluates code and
so on.

If the exception module implements the optional `c:blame/2`
callback, it will be invoked to perform the computation.


### blame_mfa(module, function, args)
*(since 1.5.0)* 
```elixir
@spec blame_mfa(module(), function :: atom(), args :: [term()]) ::
  {:ok, :def | :defp | :defmacro | :defmacrop,
   [{args :: [term()], guards :: [term()]}]}
  | :error
```

Blames the invocation of the given module, function and arguments.

This function will retrieve the available clauses from bytecode
and evaluate them against the given arguments. The clauses are
returned as a list of `{args, guards}` pairs where each argument
and each top-level condition in a guard separated by `and`/`or`
is wrapped in a tuple with blame metadata.

This function returns either `{:ok, definition, clauses}` or `:error`.
Where `definition` is `:def`, `:defp`, `:defmacro` or `:defmacrop`.


### format(kind, payload, stacktrace \\ [])

```elixir
@spec format(kind(), any(), stacktrace()) :: String.t()
```

Normalizes and formats throw/errors/exits and stacktraces.

It relies on `format_banner/3` and `format_stacktrace/1`
to generate the final format.

If `kind` is `{:EXIT, pid}`, it does not generate a stacktrace,
as such exits are retrieved as messages without stacktraces.


### format_banner(kind, exception, stacktrace \\ [])

```elixir
@spec format_banner(kind(), any(), stacktrace()) :: String.t()
```

Normalizes and formats any throw/error/exit.

The message is formatted and displayed in the same
format as used by Elixir's CLI.

The third argument is the stacktrace which is used to enrich
a normalized error with more information. It is only used when
the kind is an error.


### format_exit(reason)

```elixir
@spec format_exit(any()) :: String.t()
```

Formats an exit. It returns a string.

Often there are errors/exceptions inside exits. Exits are often
wrapped by the caller and provide stacktraces too. This function
formats exits in a way to nicely show the exit reason, caller
and stacktrace.


### format_fa(fun, arity)

```elixir
@spec format_fa((... -&gt; any()), arity()) :: String.t()
```

Receives an anonymous function and arity and formats it as
shown in stacktraces. The arity may also be a list of arguments.

#### Examples

    Exception.format_fa(fn -> nil end, 1)
    #=> "#Function<...>/1"


### format_file_line(file, line, suffix \\ &quot;&quot;)

```elixir
@spec format_file_line(String.t() | nil, non_neg_integer() | nil, String.t()) ::
  String.t()
```

Formats the given `file` and `line` as shown in stacktraces.

If any of the values are `nil`, they are omitted.

#### Examples

    iex> Exception.format_file_line("foo", 1)
    "foo:1:"
    
    iex> Exception.format_file_line("foo", nil)
    "foo:"
    
    iex> Exception.format_file_line(nil, nil)
    ""


### format_file_line_column(file, line, column, suffix \\ &quot;&quot;)

```elixir
@spec format_file_line_column(
  String.t() | nil,
  non_neg_integer() | nil,
  non_neg_integer() | nil,
  String.t()
) :: String.t()
```

Formats the given `file`, `line`, and `column` as shown in stacktraces.

If any of the values are `nil`, they are omitted.

#### Examples

    iex> Exception.format_file_line_column("foo", 1, 2)
    "foo:1:2:"
    
    iex> Exception.format_file_line_column("foo", 1, nil)
    "foo:1:"
    
    iex> Exception.format_file_line_column("foo", nil, nil)
    "foo:"
    
    iex> Exception.format_file_line_column("foo", nil, 2)
    "foo:"
    
    iex> Exception.format_file_line_column(nil, nil, nil)
    ""


### format_mfa(module, fun, arity)

```elixir
@spec format_mfa(module(), atom(), arity_or_args()) :: String.t()
```

Receives a module, fun and arity and formats it
as shown in stacktraces. The arity may also be a list
of arguments.

#### Examples

    iex> Exception.format_mfa(Foo, :bar, 1)
    "Foo.bar/1"
    
    iex> Exception.format_mfa(Foo, :bar, [])
    "Foo.bar()"
    
    iex> Exception.format_mfa(nil, :bar, [])
    "nil.bar()"

Anonymous functions are reported as -func/arity-anonfn-count-,
where func is the name of the enclosing function. Convert to
"anonymous fn in func/arity"


### format_stacktrace(trace \\ nil)

```elixir
@spec format_stacktrace(stacktrace() | nil) :: String.t()
```

Formats the stacktrace.

A stacktrace must be given as an argument. If not, the stacktrace
is retrieved from `Process.info/2`.


### format_stacktrace_entry(entry)

```elixir
@spec format_stacktrace_entry(stacktrace_entry()) :: String.t()
```

Receives a stacktrace entry and formats it into a string.


### message(exception)

```elixir
@spec message(t()) :: String.t()
```

Gets the message for an `exception`.


### normalize(kind, payload, stacktrace \\ [])

```elixir
@spec normalize(:error, any(), stacktrace()) :: t()
@spec normalize(non_error_kind(), payload, stacktrace()) :: payload when payload: var
```

Normalizes an exception, converting Erlang exceptions
to Elixir exceptions.

It takes the `kind` spilled by `catch` as an argument and
normalizes only `:error`, returning the untouched payload
for others.

The third argument is the stacktrace which is used to enrich
a normalized error with more information. It is only used when
the kind is an error.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
