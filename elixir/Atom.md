# Atom 
(Elixir v1.18.0-dev)

Atoms are constants whose values are their own name.

They are often useful to enumerate over distinct values, such as:

    iex> :apple
    :apple
    iex> :orange
    :orange
    iex> :watermelon
    :watermelon

Atoms are equal if their names are equal.

    iex> :apple == :apple
    true
    iex> :apple == :orange
    false

Often they are used to express the state of an operation, by using
values such as `:ok` and `:error`.

The booleans `true` and `false` are also atoms:

    iex> true == :true
    true
    iex> is_atom(false)
    true
    iex> is_boolean(:false)
    true

Elixir allows you to skip the leading `:` for the atoms `false`, `true`,
and `nil`.

Atoms must be composed of Unicode characters such as letters, numbers,
underscore, and `@`. If the keyword has a character that does not
belong to the category above, such as spaces, you can wrap it in
quotes:

    iex> :"this is an atom with spaces"
    :"this is an atom with spaces"

## Functions

### to_charlist(atom)

```elixir
@spec to_charlist(atom()) :: charlist()
```

Converts an atom to a charlist.

Inlined by the compiler.

#### Examples

    iex> Atom.to_charlist(:"An atom")
    ~c"An atom"

### to_string(atom)

```elixir
@spec to_string(atom()) :: String.t()
```

Converts an atom to a string.

Inlined by the compiler.

#### Examples

    iex> Atom.to_string(:foo)
    "foo"



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
