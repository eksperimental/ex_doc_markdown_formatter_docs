# Record 
(Elixir v1.18.0-dev)

Module to work with, define, and import records.

Records are simply tuples where the first element is an atom:

    iex> Record.is_record({User, "john", 27})
    true

This module provides conveniences for working with records at
compilation time, where compile-time field names are used to
manipulate the tuples, providing fast operations on top of
the tuples' compact structure.

In Elixir, records are used mostly in two situations:

1. to work with short, internal data
2. to interface with Erlang records

The macros `defrecord/3` and `defrecordp/3` can be used to create records
while `extract/2` and `extract_all/1` can be used to extract records from
Erlang files.

## Types

Types can be defined for tuples with the `record/2` macro (only available in
typespecs). This macro will expand to a tuple as seen in the example below:

    defmodule MyModule do
      require Record
      Record.defrecord(:user, name: "john", age: 25)
    
      @type user :: record(:user, name: String.t(), age: integer)
      # expands to: "@type user :: {:user, String.t(), integer}"
    end

## Reflection

The record tag and its fields are stored as metadata in the "Docs" chunk
of the record definition macro. You can retrieve the documentation for
a module by calling `Code.fetch_docs/1`.

## Guards

### is_record(data)
*(macro)* 


Checks if the given `data` is a record.

This is implemented as a macro so it can be used in guard clauses.

#### Examples

    Record.is_record({User, "john", 27})
    #=> true
    
    Record.is_record({})
    #=> false

### is_record(data, kind)
*(macro)* 


Checks if the given `data` is a record of kind `kind`.

This is implemented as a macro so it can be used in guard clauses.

#### Examples

    iex> record = {User, "john", 27}
    iex> Record.is_record(record, User)
    true

## Functions

### defrecord(name, tag \\ nil, kv)
*(macro)* 


Defines a set of macros to create, access, and pattern match
on a record.

The name of the generated macros will be `name` (which has to be an
atom). `tag` is also an atom and is used as the "tag" for the record (i.e.,
the first element of the record tuple); by default (if `nil`), it's the same
as `name`. `kv` is a keyword list of `name: default_value` fields for the
new record.

The following macros are generated:

- `name/0` to create a new record with default values for all fields
- `name/1` to create a new record with the given fields and values,
  to get the zero-based index of the given field in a record or to
  convert the given record to a keyword list
- `name/2` to update an existing record with the given fields and values
  or to access a given field in a given record

All these macros are public macros (as defined by `defmacro`).

See the "Examples" section for examples on how to use these macros.

#### Examples

    defmodule User do
      require Record
      Record.defrecord(:user, name: "meg", age: "25")
    end

In the example above, a set of macros named `user` but with different
arities will be defined to manipulate the underlying record.

    # Import the module to make the user macros locally available
    import User
    
    # To create records
    record = user()        #=> {:user, "meg", 25}
    record = user(age: 26) #=> {:user, "meg", 26}
    
    # To get a field from the record
    user(record, :name) #=> "meg"
    
    # To update the record
    user(record, age: 26) #=> {:user, "meg", 26}
    
    # To get the zero-based index of the field in record tuple
    # (index 0 is occupied by the record "tag")
    user(:name) #=> 1
    
    # Convert a record to a keyword list
    user(record) #=> [name: "meg", age: 26]

The generated macros can also be used in order to pattern match on records and
to bind variables during the match:

    record = user() #=> {:user, "meg", 25}
    
    user(name: name) = record
    name #=> "meg"

By default, Elixir uses the record name as the first element of the tuple (the "tag").
However, a different tag can be specified when defining a record,
as in the following example, in which we use `Customer` as the second argument of `defrecord/3`:

    defmodule User do
      require Record
      Record.defrecord(:user, Customer, name: nil)
    end
    
    require User
    User.user() #=> {Customer, nil}

#### Defining extracted records with anonymous functions in the values

If a record defines an anonymous function in the default values, an
`ArgumentError` will be raised. This can happen unintentionally when defining
a record after extracting it from an Erlang library that uses anonymous
functions for defaults.

    Record.defrecord(:my_rec, Record.extract(...))
    ** (ArgumentError) invalid value for record field fun_field,
        cannot escape #Function<12.90072148/2 in :erl_eval.expr/5>.

To work around this error, redefine the field with your own \&M.f/a function,
like so:

    defmodule MyRec do
      require Record
      Record.defrecord(:my_rec, Record.extract(...) |> Keyword.merge(fun_field: &__MODULE__.foo/2))
      def foo(bar, baz), do: IO.inspect({bar, baz})
    end

### defrecordp(name, tag \\ nil, kv)
*(macro)* 


Same as `defrecord/3` but generates private macros.

### extract(name, opts)

```elixir
@spec extract(
  name :: atom(),
  keyword()
) :: keyword()
```

Extracts record information from an Erlang file.

Returns a quoted expression containing the fields as a list
of tuples.

`name`, which is the name of the extracted record, is expected to be an atom
*at compile time*.

#### Options

This function requires one of the following options, which are exclusive to each
other (i.e., only one of them can be used in the same call):

- `:from` - (binary representing a path to a file) path to the Erlang file
  that contains the record definition to extract; with this option, this
  function uses the same path lookup used by the `-include` attribute used in
  Erlang modules.

- `:from_lib` - (binary representing a path to a file) path to the Erlang
  file that contains the record definition to extract; with this option,
  this function uses the same path lookup used by the `-include_lib`
  attribute used in Erlang modules.

It additionally accepts the following optional, non-exclusive options:

- `:includes` - (a list of directories as binaries) if the record being
  extracted depends on relative includes, this option allows developers
  to specify the directory where those relative includes exist.

- `:macros` - (keyword list of macro names and values) if the record
  being extracted depends on the values of macros, this option allows
  the value of those macros to be set.

These options are expected to be literals (including the binary values) at
compile time.

#### Examples

    iex> Record.extract(:file_info, from_lib: "kernel/include/file.hrl")
    [
      size: :undefined,
      type: :undefined,
      access: :undefined,
      atime: :undefined,
      mtime: :undefined,
      ctime: :undefined,
      mode: :undefined,
      links: :undefined,
      major_device: :undefined,
      minor_device: :undefined,
      inode: :undefined,
      uid: :undefined,
      gid: :undefined
    ]

### extract_all(opts)

```elixir
@spec extract_all(keyword()) :: [{name :: atom(), keyword()}]
```

Extracts all records information from an Erlang file.

Returns a keyword list of `{record_name, fields}` tuples where `record_name`
is the name of an extracted record and `fields` is a list of `{field, value}`
tuples representing the fields for that record.

#### Options

Accepts the same options as listed for `Record.extract/2`.



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
