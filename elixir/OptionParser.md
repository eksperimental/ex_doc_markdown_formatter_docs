# OptionParser 
(Elixir v1.18.0-dev)

Functions for parsing command line arguments.

When calling a command, it's possible to pass command line options
to modify what the command does. In this documentation, those are
called "switches", in other situations they may be called "flags"
or simply "options". A switch can be given a value, also called an
"argument".

The main function in this module is `parse/2`, which parses a list
of command line options and arguments into a keyword list:

    iex> OptionParser.parse(["--debug"], strict: [debug: :boolean])
    {[debug: true], [], []}

`OptionParser` provides some conveniences out of the box,
such as aliases and automatic handling of negation switches.

The `parse_head/2` function is an alternative to `parse/2`
which stops parsing as soon as it finds a value that is not
a switch nor a value for a previous switch.

This module also provides low-level functions, such as `next/2`,
for parsing switches manually, as well as `split/1` and `to_argv/1`
for parsing from and converting switches to strings.


## Types

### argv()

```elixir
@type argv() :: [String.t()]
```



### errors()

```elixir
@type errors() :: [{String.t(), String.t() | nil}]
```



### options()

```elixir
@type options() :: [
  switches: keyword(),
  strict: keyword(),
  aliases: keyword(),
  allow_nonexistent_atoms: boolean(),
  return_separator: boolean()
]
```



### parsed()

```elixir
@type parsed() :: keyword()
```



## Functions

### next(argv, opts \\ [])

```elixir
@spec next(argv(), options()) ::
  {:ok, key :: atom(), value :: term(), argv()}
  | {:invalid, String.t(), String.t() | nil, argv()}
  | {:undefined, String.t(), String.t() | nil, argv()}
  | {:error, argv()}
```

Low-level function that parses one option.

It accepts the same options as `parse/2` and `parse_head/2`
as both functions are built on top of this function. This function
may return:

- `{:ok, key, value, rest}` - the option `key` with `value` was
  successfully parsed

- `{:invalid, key, value, rest}` - the option `key` is invalid with `value`
  (returned when the value cannot be parsed according to the switch type)

- `{:undefined, key, value, rest}` - the option `key` is undefined
  (returned in strict mode when the switch is unknown or on nonexistent atoms)

- `{:error, rest}` - there are no switches at the head of the given `argv`


### parse(argv, opts \\ [])

```elixir
@spec parse(argv(), options()) :: {parsed(), argv(), errors()}
```

Parses `argv` into a keyword list.

It returns a three-element tuple with the form `{parsed, args, invalid}`, where:

- `parsed` is a keyword list of parsed switches with `{switch_name, value}`
  tuples in it; `switch_name` is the atom representing the switch name while
  `value` is the value for that switch parsed according to `opts` (see the
  "Examples" section for more information)
- `args` is a list of the remaining arguments in `argv` as strings
- `invalid` is a list of invalid options as `{option_name, value}` where
  `option_name` is the raw option and `value` is `nil` if the option wasn't
  expected or the string value if the value didn't have the expected type for
  the corresponding option

Elixir converts switches to underscored atoms, so `--source-path` becomes
`:source_path`. This is done to better suit Elixir conventions. However, this
means that switches can't contain underscores and switches that do contain
underscores are always returned in the list of invalid switches.

When parsing, it is common to list switches and their expected types:

    iex> OptionParser.parse(["--debug"], strict: [debug: :boolean])
    {[debug: true], [], []}
    
    iex> OptionParser.parse(["--source", "lib"], strict: [source: :string])
    {[source: "lib"], [], []}
    
    iex> OptionParser.parse(
    ...>   ["--source-path", "lib", "test/enum_test.exs", "--verbose"],
    ...>   strict: [source_path: :string, verbose: :boolean]
    ...> )
    {[source_path: "lib", verbose: true], ["test/enum_test.exs"], []}

We will explore the valid switches and operation modes of option parser below.

#### Options

The following options are supported:

- `:switches` or `:strict` - see the "Switch definitions" section below
- `:allow_nonexistent_atoms` - see the "Parsing unknown switches" section below
- `:aliases` - see the "Aliases" section below
- `:return_separator` - see the "Return separator" section below

#### Switch definitions

Switches can be specified via one of two options:

- `:strict` - defines strict switches and their types. Any switch
  in `argv` that is not specified in the list is returned in the
  invalid options list. This is the preferred way to parse options.

- `:switches` - defines switches and their types. This function
  still attempts to parse switches that are not in this list.

Both these options accept a keyword list where the key is an atom
defining the name of the switch and value is the `type` of the
switch (see the "Types" section below for more information).

Note that you should only supply the `:switches` or the `:strict` option.
If you supply both, an `ArgumentError` exception will be raised.

##### Types

Switches parsed by `OptionParser` may take zero or one arguments.

The following switches types take no arguments:

- `:boolean` - sets the value to `true` when given (see also the
  "Negation switches" section below)
- `:count` - counts the number of times the switch is given

The following switches take one argument:

- `:integer` - parses the value as an integer
- `:float` - parses the value as a float
- `:string` - parses the value as a string

If a switch can't be parsed according to the given type, it is
returned in the invalid options list.

##### Modifiers

Switches can be specified with modifiers, which change how
they behave. The following modifiers are supported:

- `:keep` - keeps duplicate elements instead of overriding them;
  works with all types except `:count`. Specifying `switch_name: :keep`
  assumes the type of `:switch_name` will be `:string`.

To use `:keep` with a type other than `:string`, use a list as the type
for the switch. For example: `[foo: [:integer, :keep]]`.

##### Negation switches

In case a switch `SWITCH` is specified to have type `:boolean`, it may be
passed as `--no-SWITCH` as well which will set the option to `false`:

    iex> OptionParser.parse(["--no-op", "path/to/file"], switches: [op: :boolean])
    {[op: false], ["path/to/file"], []}

##### Parsing unknown switches

When the `:switches` option is given, `OptionParser` will attempt to parse
unknown switches.

Switches without an argument will be set to `true`:

    iex> OptionParser.parse(["--debug"], switches: [key: :string])
    {[debug: true], [], []}

Even though we haven't specified `--debug` in the list of switches, it is part
of the returned options. The same happens for switches followed by another switch:

    iex> OptionParser.parse(["--debug", "--ok"], switches: [])
    {[debug: true, ok: true], [], []}

Switches followed by a value will be assigned the value, as a string:

    iex> OptionParser.parse(["--debug", "value"], switches: [key: :string])
    {[debug: "value"], [], []}

Since we cannot assert the type of the switch value, it is preferred to use the
`:strict` option that accepts only known switches and always verify their types.

If you do want to parse unknown switches, remember that Elixir converts switches
to atoms. Since atoms are not garbage-collected, to avoid creating new ones,
OptionParser by default only parses switches that translate to existing atoms.
The code below discards the `--option-parser-example` switch because the
`:option_parser_example` atom is never used anywhere:

    iex> OptionParser.parse(["--option-parser-example"], switches: [])
    {[], [], []}

If a switch corresponds to an existing Elixir atom, whether from your
code, a dependency or from Elixir itself, it will be accepted. However,
it is best to not rely on external code, and always define the atoms
you want to parse in the same module that calls `OptionParser` itself,
as direct arguments to the `:switches` or `:strict` options.

If you would like to parse all switches, regardless if they exist or not,
you can force creation of atoms by passing `allow_nonexistent_atoms: true`
as option. Use this option with care. It is only useful when you are building
command-line applications that receive dynamically-named arguments and must
be avoided in long-running systems.

#### Aliases

A set of aliases can be specified in the `:aliases` option:

    iex> OptionParser.parse(["-d"], aliases: [d: :debug], strict: [debug: :boolean])
    {[debug: true], [], []}

#### Examples

Here are some examples of working with different types and modifiers:

    iex> OptionParser.parse(["--unlock", "path/to/file"], strict: [unlock: :boolean])
    {[unlock: true], ["path/to/file"], []}
    
    iex> OptionParser.parse(
    ...>   ["--unlock", "--limit", "0", "path/to/file"],
    ...>   strict: [unlock: :boolean, limit: :integer]
    ...> )
    {[unlock: true, limit: 0], ["path/to/file"], []}
    
    iex> OptionParser.parse(["--limit", "3"], strict: [limit: :integer])
    {[limit: 3], [], []}
    
    iex> OptionParser.parse(["--limit", "xyz"], strict: [limit: :integer])
    {[], [], [{"--limit", "xyz"}]}
    
    iex> OptionParser.parse(["--verbose"], switches: [verbose: :count])
    {[verbose: 1], [], []}
    
    iex> OptionParser.parse(["-v", "-v"], aliases: [v: :verbose], strict: [verbose: :count])
    {[verbose: 2], [], []}
    
    iex> OptionParser.parse(["--unknown", "xyz"], strict: [])
    {[], ["xyz"], [{"--unknown", nil}]}
    
    iex> OptionParser.parse(
    ...>   ["--limit", "3", "--unknown", "xyz"],
    ...>   switches: [limit: :integer]
    ...> )
    {[limit: 3, unknown: "xyz"], [], []}
    
    iex> OptionParser.parse(
    ...>   ["--unlock", "path/to/file", "--unlock", "path/to/another/file"],
    ...>   strict: [unlock: :keep]
    ...> )
    {[unlock: "path/to/file", unlock: "path/to/another/file"], [], []}

#### Return separator

The separator `--` implies options should no longer be processed.
By default, the separator is not returned as parts of the arguments,
but that can be changed via the `:return_separator` option:

    iex> OptionParser.parse(["--", "lib"], return_separator: true, strict: [])
    {[], ["--", "lib"], []}
    
    iex> OptionParser.parse(["--no-halt", "--", "lib"], return_separator: true, switches: [halt: :boolean])
    {[halt: false], ["--", "lib"], []}
    
    iex> OptionParser.parse(["script.exs", "--no-halt", "--", "foo"], return_separator: true, switches: [halt: :boolean])
    {[{:halt, false}], ["script.exs", "--", "foo"], []}


### parse!(argv, opts \\ [])

```elixir
@spec parse!(argv(), options()) :: {parsed(), argv()}
```

The same as `parse/2` but raises an `OptionParser.ParseError`
exception if any invalid options are given.

If there are no errors, returns a `{parsed, rest}` tuple where:

- `parsed` is the list of parsed switches (same as in `parse/2`)
- `rest` is the list of arguments (same as in `parse/2`)

#### Examples

    iex> OptionParser.parse!(["--debug", "path/to/file"], strict: [debug: :boolean])
    {[debug: true], ["path/to/file"]}
    
    iex> OptionParser.parse!(["--limit", "xyz"], strict: [limit: :integer])
    ** (OptionParser.ParseError) 1 error found!
    --limit : Expected type integer, got "xyz"
    
    iex> OptionParser.parse!(["--unknown", "xyz"], strict: [])
    ** (OptionParser.ParseError) 1 error found!
    --unknown : Unknown option
    
    iex> OptionParser.parse!(
    ...>   ["-l", "xyz", "-f", "bar"],
    ...>   switches: [limit: :integer, foo: :integer],
    ...>   aliases: [l: :limit, f: :foo]
    ...> )
    ** (OptionParser.ParseError) 2 errors found!
    -l : Expected type integer, got "xyz"
    -f : Expected type integer, got "bar"


### parse_head(argv, opts \\ [])

```elixir
@spec parse_head(argv(), options()) :: {parsed(), argv(), errors()}
```

Similar to `parse/2` but only parses the head of `argv`;
as soon as it finds a non-switch, it stops parsing.

See `parse/2` for more information.

#### Example

    iex> OptionParser.parse_head(
    ...>   ["--source", "lib", "test/enum_test.exs", "--verbose"],
    ...>   switches: [source: :string, verbose: :boolean]
    ...> )
    {[source: "lib"], ["test/enum_test.exs", "--verbose"], []}
    
    iex> OptionParser.parse_head(
    ...>   ["--verbose", "--source", "lib", "test/enum_test.exs", "--unlock"],
    ...>   switches: [source: :string, verbose: :boolean, unlock: :boolean]
    ...> )
    {[verbose: true, source: "lib"], ["test/enum_test.exs", "--unlock"], []}


### parse_head!(argv, opts \\ [])

```elixir
@spec parse_head!(argv(), options()) :: {parsed(), argv()}
```

The same as `parse_head/2` but raises an `OptionParser.ParseError`
exception if any invalid options are given.

If there are no errors, returns a `{parsed, rest}` tuple where:

- `parsed` is the list of parsed switches (same as in `parse_head/2`)
- `rest` is the list of arguments (same as in `parse_head/2`)

#### Examples

    iex> OptionParser.parse_head!(
    ...>   ["--source", "lib", "path/to/file", "--verbose"],
    ...>   switches: [source: :string, verbose: :boolean]
    ...> )
    {[source: "lib"], ["path/to/file", "--verbose"]}
    
    iex> OptionParser.parse_head!(
    ...>   ["--number", "lib", "test/enum_test.exs", "--verbose"],
    ...>   strict: [number: :integer]
    ...> )
    ** (OptionParser.ParseError) 1 error found!
    --number : Expected type integer, got "lib"
    
    iex> OptionParser.parse_head!(
    ...>   ["--verbose", "--source", "lib", "test/enum_test.exs", "--unlock"],
    ...>   strict: [verbose: :integer, source: :integer]
    ...> )
    ** (OptionParser.ParseError) 2 errors found!
    --verbose : Missing argument of type integer
    --source : Expected type integer, got "lib"


### split(string)

```elixir
@spec split(String.t()) :: argv()
```

Splits a string into `t:argv/0` chunks.

This function splits the given `string` into a list of strings in a similar
way to many shells.

#### Examples

    iex> OptionParser.split("foo bar")
    ["foo", "bar"]
    
    iex> OptionParser.split("foo \"bar baz\"")
    ["foo", "bar baz"]


### to_argv(enum, options \\ [])

```elixir
@spec to_argv(Enumerable.t(), options()) :: argv()
```

Receives a key-value enumerable and converts it to `t:argv/0`.

Keys must be atoms. Keys with `nil` value are discarded,
boolean values are converted to `--key` or `--no-key`
(if the value is `true` or `false`, respectively),
and all other values are converted using `to_string/1`.

It is advised to pass to `to_argv/2` the same set of `options`
given to `parse/2`. Some switches can only be reconstructed
correctly with the `:switches` information in hand.

#### Examples

    iex> OptionParser.to_argv(foo_bar: "baz")
    ["--foo-bar", "baz"]
    iex> OptionParser.to_argv(bool: true, bool: false, discarded: nil)
    ["--bool", "--no-bool"]

Some switches will output different values based on the switches
types:

    iex> OptionParser.to_argv([number: 2], switches: [])
    ["--number", "2"]
    iex> OptionParser.to_argv([number: 2], switches: [number: :count])
    ["--number", "--number"]




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
