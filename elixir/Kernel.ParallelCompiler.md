# Kernel.ParallelCompiler 
(Elixir v1.18.0-dev)

A module responsible for compiling and requiring files in parallel.


## Types

### error()

```elixir
@type error() :: {file :: Path.t(), Code.position(), message :: String.t()}
```



### info()

```elixir
@type info() :: %{
  runtime_warnings: [Code.diagnostic(:warning)],
  compile_warnings: [Code.diagnostic(:warning)]
}
```



### warning()

```elixir
@type warning() :: {file :: Path.t(), Code.position(), message :: String.t()}
```



## Functions

### async(fun)


This function is deprecated. Use `pmap/2` instead.
Starts a task for parallel compilation.


### compile(files, options \\ [])
*(since 1.6.0)* 
```elixir
@spec compile(
  [Path.t()],
  keyword()
) ::
  {:ok, [atom()], [warning()] | info()}
  | {:error, [error()] | [Code.diagnostic(:error)], [warning()] | info()}
```

Compiles the given files.

Those files are compiled in parallel and can automatically
detect dependencies between them. Once a dependency is found,
the current file stops being compiled until the dependency is
resolved.

It returns `{:ok, modules, warnings}` or `{:error, errors, warnings}`
by default but we recommend using `return_diagnostics: true` so it returns
diagnostics as maps as well as a map of compilation information.
The map has the shape of:

    %{
      runtime_warnings: [warning],
      compile_warnings: [warning]
    }

#### Options

- `:each_file` - for each file compiled, invokes the callback passing the
  file

- `:each_long_compilation` - for each file that takes more than a given
  timeout (see the `:long_compilation_threshold` option) to compile, invoke
  this callback passing the file as its argument

- `:each_module` - for each module compiled, invokes the callback passing
  the file, module and the module bytecode

- `:each_cycle` - after the given files are compiled, invokes this function
  that should return the following values:
  
  - `{:compile, modules, warnings}` - to continue compilation with a list of
    further modules to compile
  - `{:runtime, modules, warnings}` - to stop compilation and verify the list
    of modules because dependent modules have changed

- `:long_compilation_threshold` - the timeout (in seconds) to check for modules
  taking too long to compile. For each file that exceeds the threshold, the
  `:each_long_compilation` callback is invoked. From Elixir v1.11, only the time
  spent compiling the actual module is taken into account by the threshold, the
  time spent waiting is not considered. Defaults to `10` seconds.

- `:profile` - if set to `:time` measure the compilation time of each compilation cycle
  and group pass checker

- `:dest` - the destination directory for the BEAM files. When using `compile/2`,
  this information is only used to properly annotate the BEAM files before
  they are loaded into memory. If you want a file to actually be written to
  `dest`, use `compile_to_path/3` instead.

- `:beam_timestamp` - the modification timestamp to give all BEAM files

- `:return_diagnostics` (since v1.15.0) - returns maps with information instead of
  a list of warnings and returns diagnostics as maps instead of tuples

- `:max_concurrency` - the maximum number of files to compile in parallel.
  Setting this option to 1 will compile files sequentially.
  Defaults to the number of schedulers online, or at least 2.


### compile_to_path(files, path, options \\ [])
*(since 1.6.0)* 
```elixir
@spec compile_to_path([Path.t()], Path.t(), keyword()) ::
  {:ok, [atom()], [warning()] | info()}
  | {:error, [error()] | [Code.diagnostic(:error)], [warning()] | info()}
```

Compiles the given files and writes resulting BEAM files into path.

See `compile/2` for more information.


### pmap(collection, fun)
*(since 1.16.0)* 


Perform parallel compilation of `collection` with `fun`.

If you have a file that needs to compile other modules in parallel,
the spawned processes need to be aware of the compiler environment.
This function allows a developer to perform such tasks.


### require(files, options \\ [])
*(since 1.6.0)* 
```elixir
@spec require(
  [Path.t()],
  keyword()
) ::
  {:ok, [atom()], [warning()] | info()}
  | {:error, [error()] | [Code.diagnostic(:error)], [warning()] | info()}
```

Requires the given files in parallel.

Opposite to compile, dependencies are not attempted to be
automatically solved between files.

It returns `{:ok, modules, warnings}` or `{:error, errors, warnings}`
by default but we recommend using `return_diagnostics: true` so it returns
diagnostics as maps as well as a map of compilation information.
The map has the shape of:

    %{
      runtime_warnings: [warning],
      compile_warnings: [warning]
    }

#### Options

- `:each_file` - for each file compiled, invokes the callback passing the
  file

- `:each_module` - for each module compiled, invokes the callback passing
  the file, module and the module bytecode

- `:max_concurrency` - the maximum number of files to compile in parallel.
  Setting this option to 1 will compile files sequentially.
  Defaults to the number of schedulers online, or at least 2.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
