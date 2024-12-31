# Macro.Env 
(Elixir v1.18.0-dev)

A struct that holds compile time environment information.

The current environment can be accessed at any time as
`__ENV__/0`. Inside macros, the caller environment can be
accessed as `__CALLER__/0`.

The majority of the functions in this module are provided
for low-level tools, which need to integrate with the Elixir
compiler, such as language servers and embedded languages.
For regular usage in Elixir code and macros, you must use
the `Macro` module instead. In particular, avoid modifying
the `Macro.Env` struct directly and prefer to use high-level
constructs, such as a `import`, `aliases`, and so forth to
build your own environment. For example, to build a custom
environment, you can define a function such as:

    def make_custom_env do
      import SomeModule, only: [some_function: 2], warn: false
      alias A.B.C, warn: false
      __ENV__
    end

## Struct fields

The `Macro.Env` struct contains the following fields:

- `context` - the context of the environment; it can be `nil`
  (default context), `:guard` (inside a guard) or `:match` (inside a match)
- `context_modules` - a list of modules defined in the current context
- `file` - the current absolute file name as a binary
- `function` - a tuple as `{atom, integer}`, where the first
  element is the function name and the second its arity; returns
  `nil` if not inside a function
- `line` - the current line as an integer
- `module` - the current module name

The following fields are private to Elixir's macro expansion mechanism and
must not be accessed directly:

- `aliases`
- `functions`
- `macro_aliases`
- `macros`
- `lexical_tracker`
- `requires`
- `tracers`
- `versioned_vars`


## Types

### context()

```elixir
@type context() :: :match | :guard | nil
```



### context_modules()

```elixir
@type context_modules() :: [module()]
```



### file()

```elixir
@type file() :: binary()
```



### line()

```elixir
@type line() :: non_neg_integer()
```



### name_arity()

```elixir
@type name_arity() :: {atom(), arity()}
```



### t()

```elixir
@type t() :: %Macro.Env{
  aliases: aliases(),
  context: context(),
  context_modules: context_modules(),
  file: file(),
  function: name_arity() | nil,
  functions: functions(),
  lexical_tracker: lexical_tracker(),
  line: line(),
  macro_aliases: macro_aliases(),
  macros: macros(),
  module: module(),
  requires: requires(),
  tracers: tracers(),
  versioned_vars: versioned_vars()
}
```



### variable()

```elixir
@type variable() :: {atom(), atom() | term()}
```



## Functions

### define_alias(env, meta, module, opts \\ [])
*(since 1.17.0)* 
```elixir
@spec define_alias(t(), Macro.metadata(), module(), keyword()) ::
  {:ok, t()} | {:error, String.t()}
```

Defines the given `as` an alias to `module` in the environment.

This is used by tools which need to mimic the Elixir compiler.
The appropriate `:alias` compiler tracing event will be emitted.

#### Additional options

It accepts the same options as `Kernel.SpecialForm.alias/2` plus:

- `:trace` - when set to `false`, it disables compilation tracers and
  lexical tracker. This option must only be used by language servers and
  other tools that need to introspect code without affecting how it is compiled.
  Disabling tracer inside macros or regular code expansion is extremely
  discouraged as it blocks the compiler from accurately tracking dependencies

#### Examples

    iex> env = __ENV__
    iex> Macro.Env.expand_alias(env, [], [:Baz])
    :error
    iex> {:ok, env} = Macro.Env.define_alias(env, [line: 10], Foo.Bar, as: Baz)
    iex> Macro.Env.expand_alias(env, [], [:Baz])
    {:alias, Foo.Bar}
    iex> Macro.Env.expand_alias(env, [], [:Baz, :Bat])
    {:alias, Foo.Bar.Bat}

If no `:as` option is given, the alias will be inferred from the module:

    iex> env = __ENV__
    iex> {:ok, env} = Macro.Env.define_alias(env, [line: 10], Foo.Bar)
    iex> Macro.Env.expand_alias(env, [], [:Bar])
    {:alias, Foo.Bar}

If it is not possible to infer one, an error is returned:

    iex> Macro.Env.define_alias(__ENV__, [line: 10], :an_atom)
    {:error,
     "alias cannot be inferred automatically for module: :an_atom, " <>
       "please use the :as option. Implicit aliasing is only supported with Elixir modules"}


### define_import(env, meta, module, opts \\ [])
*(since 1.17.0)* 
```elixir
@spec define_import(t(), Macro.metadata(), module(), keyword()) ::
  {:ok, t()} | {:error, String.t()}
```

Defines the given `module` as imported in the environment.

It assumes `module` is available. This is used by tools which
need to mimic the Elixir compiler. The appropriate `:import`
compiler tracing event will be emitted.

#### Additional options

It accepts the same options as `Kernel.SpecialForm.import/2` plus:

- `:emit_warnings` - emit warnings found when defining imports

- `:trace` - when set to `false`, it disables compilation tracers and
  lexical tracker. This option must only be used by language servers and
  other tools that need to introspect code without affecting how it is compiled.
  Disabling tracer inside macros or regular code expansion is extremely
  discouraged as it blocks the compiler from accurately tracking dependencies

- `:info_callback` - a function to use instead of `c:Module.__info__/1`.
  The function will be invoked with `:functions` or `:macros` argument.
  It has to return a list of `{function, arity}` key value pairs.
  If it fails, it defaults to using module metadata based on `module_info/1`.

#### Examples

    iex> env = __ENV__
    iex> Macro.Env.lookup_import(env, {:flatten, 1})
    []
    iex> {:ok, env} = Macro.Env.define_import(env, [line: 10], List)
    iex> Macro.Env.lookup_import(env, {:flatten, 1})
    [{:function, List}]

It accepts the same options as `Kernel.SpecialForm.import/2`:

    iex> env = __ENV__
    iex> Macro.Env.lookup_import(env, {:is_odd, 1})
    []
    iex> {:ok, env} = Macro.Env.define_import(env, [line: 10], Integer, only: :macros)
    iex> Macro.Env.lookup_import(env, {:is_odd, 1})
    [{:macro, Integer}]

#### Info callback override

    iex> env = __ENV__
    iex> Macro.Env.lookup_import(env, {:flatten, 1})
    []
    iex> {:ok, env} = Macro.Env.define_import(env, [line: 10], SomeModule, [info_callback: fn :functions -> [{:flatten, 1}]; :macros -> [{:some, 2}]; end])
    iex> Macro.Env.lookup_import(env, {:flatten, 1})
    [{:function, SomeModule}]
    iex> Macro.Env.lookup_import(env, {:some, 2})
    [{:macro, SomeModule}]


### define_require(env, meta, module, opts \\ [])
*(since 1.17.0)* 


Defines the given `module` as required in the environment.

It does not check or assert the module is available.
This is used by tools which need to mimic the Elixir compiler.
The appropriate `:require` compiler tracing event will be emitted.

#### Additional options

It accepts the same options as `Kernel.SpecialForm.require/2` plus:

- `:trace` - when set to `false`, it disables compilation tracers and
  lexical tracker. This option must only be used by language servers and
  other tools that need to introspect code without affecting how it is compiled.
  Disabling tracer inside macros or regular code expansion is extremely
  discouraged as it blocks the compiler from accurately tracking dependencies

#### Examples

    iex> env = __ENV__
    iex> Macro.Env.required?(env, Integer)
    false
    iex> {:ok, env} = Macro.Env.define_require(env, [line: 10], Integer)
    iex> Macro.Env.required?(env, Integer)
    true

If the `:as` option is given, it will also define an alias:

    iex> env = __ENV__
    iex> {:ok, env} = Macro.Env.define_require(env, [line: 10], Foo.Bar, as: Baz)
    iex> Macro.Env.expand_alias(env, [], [:Baz])
    {:alias, Foo.Bar}


### expand_alias(env, meta, list, opts \\ [])
*(since 1.17.0)* 
```elixir
@spec expand_alias(t(), keyword(), [atom()], keyword()) :: {:alias, atom()} | :error
```

Expands an alias given by the alias segments.

It returns `{:alias, alias}` if the segments is a list
of atoms and an alias was found. Returns `:error` otherwise.

This expansion may emit the `:alias_expansion` trace event
but it does not emit the `:alias_reference` one.

#### Options

- `:trace` - when set to `false`, it disables compilation tracers and
  lexical tracker. This option must only be used by language servers and
  other tools that need to introspect code without affecting how it is compiled.
  Disabling tracer inside macros or regular code expansion is extremely
  discouraged as it blocks the compiler from accurately tracking dependencies

#### Examples

    iex> alias List, as: MyList
    iex> Macro.Env.expand_alias(__ENV__, [], [:MyList])
    {:alias, List}
    iex> Macro.Env.expand_alias(__ENV__, [], [:MyList, :Nested])
    {:alias, List.Nested}

If there is no alias or the alias starts with `Elixir.`
(which disables aliasing), then `:error` is returned:

    iex> alias List, as: MyList
    iex> Macro.Env.expand_alias(__ENV__, [], [:Elixir, MyList])
    :error
    iex> Macro.Env.expand_alias(__ENV__, [], [:AnotherList])
    :error


### expand_import(env, meta, name, arity, opts \\ [])
*(since 1.17.0)* 
```elixir
@spec expand_import(t(), keyword(), atom(), arity(), keyword()) ::
  {:macro, module(), (Macro.metadata(), args :: [Macro.t()] -&gt; Macro.t())}
  | {:function, module(), atom()}
  | {:error, :not_found | {:conflict, module()} | {:ambiguous, [module()]}}
```

Expands an import given by `name` and `arity`.

If the import points to a macro, it returns a tuple
with the module and a function that expands the macro.
The function expects the metadata to be attached to the
expansion and the arguments of the macro.

If the import points to a function, it returns a tuple
with the module and the function name.

If any import is found, the appropriate compiler tracing
event will be emitted.

Otherwise returns `{:error, reason}`.

#### Options

- `:allow_locals` - when set to `false`, it does not attempt to capture
  local macros defined in the current module in `env`

- `:check_deprecations` - when set to `false`, does not check for deprecations
  when expanding macros

- `:trace` - when set to `false`, it disables compilation tracers and
  lexical tracker. This option must only be used by language servers and
  other tools that need to introspect code without affecting how it is compiled.
  Disabling tracer inside macros or regular code expansion is extremely
  discouraged as it blocks the compiler from accurately tracking dependencies


### expand_require(env, meta, module, name, arity, opts \\ [])
*(since 1.17.0)* 
```elixir
@spec expand_require(t(), keyword(), module(), atom(), arity(), keyword()) ::
  {:macro, module(), (Macro.metadata(), args :: [Macro.t()] -&gt; Macro.t())}
  | :error
```

Expands a require given by `module`, `name`, and `arity`.

If the require points to a macro and the module has been
required, it returns a tuple with the module and a function
that expands the macro. The function expects the metadata
to be attached to the expansion and the arguments of the macro.
The appropriate `:remote_macro` compiler tracing event will
be emitted if a macro is found (note a `:remote_function`
event is not emitted in `:error` cases).

Otherwise returns `:error`.

#### Options

- `:check_deprecations` - when set to `false`, does not check for deprecations
  when expanding macros

- `:trace` - when set to `false`, it disables compilation tracers and
  lexical tracker. This option must only be used by language servers and
  other tools that need to introspect code without affecting how it is compiled.
  Disabling tracer inside macros or regular code expansion is extremely
  discouraged as it blocks the compiler from accurately tracking dependencies


### has_var?(env, var)
*(since 1.7.0)* 
```elixir
@spec has_var?(t(), variable()) :: boolean()
```

Checks if a variable belongs to the environment.

#### Examples

    iex> x = 13
    iex> x
    13
    iex> Macro.Env.has_var?(__ENV__, {:x, nil})
    true
    iex> Macro.Env.has_var?(__ENV__, {:unknown, nil})
    false


### in_guard?(env)

```elixir
@spec in_guard?(t()) :: boolean()
```

Returns whether the compilation environment is currently
inside a guard.


### in_match?(env)

```elixir
@spec in_match?(t()) :: boolean()
```

Returns whether the compilation environment is currently
inside a match clause.


### location(env)

```elixir
@spec location(t()) :: keyword()
```

Returns a keyword list containing the file and line
information as keys.


### lookup_alias_as(env, atom)
*(since 1.15.0)* 
```elixir
@spec lookup_alias_as(t(), atom()) :: [atom()]
```

Returns the names of any aliases for the given module or atom.

#### Examples

    iex> alias Foo.Bar
    iex> Bar
    Foo.Bar
    iex> Macro.Env.lookup_alias_as(__ENV__, Foo.Bar)
    [Elixir.Bar]
    iex> alias Foo.Bar, as: Baz
    iex> Baz
    Foo.Bar
    iex> Macro.Env.lookup_alias_as(__ENV__, Foo.Bar)
    [Elixir.Bar, Elixir.Baz]
    iex> Macro.Env.lookup_alias_as(__ENV__, Unknown)
    []


### lookup_import(env, name_arity)
*(since 1.13.0)* 
```elixir
@spec lookup_import(t(), name_arity()) :: [{:function | :macro, module()}]
```

Returns the modules from which the given `{name, arity}` was
imported.

It returns a list of two element tuples in the shape of
`{:function | :macro, module}`. The elements in the list
are in no particular order and the order is not guaranteed.

> #### Use only for introspection {: .warning}
> 
> This function does not emit compiler tracing events,
> which may block the compiler from correctly tracking
> dependencies. Use this function for reflection purposes
> but to do not use it to expand imports into qualified
> calls. Instead, use `expand_import/5`.

#### Examples

    iex> Macro.Env.lookup_import(__ENV__, {:duplicate, 2})
    []
    iex> import Tuple, only: [duplicate: 2], warn: false
    iex> Macro.Env.lookup_import(__ENV__, {:duplicate, 2})
    [{:function, Tuple}]
    iex> import List, only: [duplicate: 2], warn: false
    iex> Macro.Env.lookup_import(__ENV__, {:duplicate, 2})
    [{:function, List}, {:function, Tuple}]
    
    iex> Macro.Env.lookup_import(__ENV__, {:def, 1})
    [{:macro, Kernel}]


### prepend_tracer(env, tracer)
*(since 1.13.0)* 
```elixir
@spec prepend_tracer(t(), module()) :: t()
```

Prepend a tracer to the list of tracers in the environment.

#### Examples

    Macro.Env.prepend_tracer(__ENV__, MyCustomTracer)


### prune_compile_info(env)
*(since 1.14.0)* 
```elixir
@spec prune_compile_info(t()) :: t()
```

Prunes compile information from the environment.

This happens when the environment is captured at compilation
time, for example, in the module body, and then used to
evaluate code after the module has been defined.


### required?(env, module)
*(since 1.13.0)* 
```elixir
@spec required?(t(), module()) :: boolean()
```

Returns `true` if the given module has been required.

#### Examples

    iex> Macro.Env.required?(__ENV__, Integer)
    false
    iex> require Integer
    iex> Macro.Env.required?(__ENV__, Integer)
    true
    
    iex> Macro.Env.required?(__ENV__, Kernel)
    true


### stacktrace(env)

```elixir
@spec stacktrace(t()) :: list()
```

Returns the environment stacktrace.


### to_guard(env)
*(since 1.17.0)* 
```elixir
@spec to_guard(t()) :: t()
```

Returns an environment in the guard context.


### to_match(env)

```elixir
@spec to_match(t()) :: t()
```

Returns an environment in the match context.


### vars(env)
*(since 1.7.0)* 
```elixir
@spec vars(t()) :: [variable()]
```

Returns a list of variables in the current environment.

Each variable is identified by a tuple of two elements,
where the first element is the variable name as an atom
and the second element is its context, which may be an
atom or an integer.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
