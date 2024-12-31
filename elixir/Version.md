# Version 
(Elixir v1.18.0-dev)

Functions for parsing and matching versions against requirements.

A version is a string in a specific format or a `Version`
generated after parsing via `Version.parse/1`.

Although Elixir projects are not required to follow SemVer,
they must follow the format outlined on [SemVer 2.0 schema](https://semver.org/).

## Versions

In a nutshell, a version is represented by three numbers:

    MAJOR.MINOR.PATCH

Pre-releases are supported by optionally appending a hyphen and a series of
period-separated identifiers immediately following the patch version.
Identifiers consist of only ASCII alphanumeric characters and hyphens (`[0-9A-Za-z-]`):

    "1.0.0-alpha.3"

Build information can be added by appending a plus sign and a series of
dot-separated identifiers immediately following the patch or pre-release version.
Identifiers consist of only ASCII alphanumeric characters and hyphens (`[0-9A-Za-z-]`):

    "1.0.0-alpha.3+20130417140000.amd64"

## Requirements

Requirements allow you to specify which versions of a given
dependency you are willing to work against. Requirements support the common
comparison operators such as `>`, `>=`, `<`, `<=`, and `==` that work as one
would expect, and additionally the special operator `~>` described in detail
further below.

    # Only version 2.0.0
    "== 2.0.0"
    
    # Anything later than 2.0.0
    "> 2.0.0"

Requirements also support `and` and `or` for complex conditions:

    # 2.0.0 and later until 2.1.0
    ">= 2.0.0 and < 2.1.0"

Since the example above is such a common requirement, it can
be expressed as:

    "~> 2.0.0"

`~>` will never include pre-release versions of its upper bound,
regardless of the usage of the `:allow_pre` option, or whether the operand
is a pre-release version. It can also be used to set an upper bound on only the major
version part. See the table below for `~>` requirements and
their corresponding translations.

`~>`           | Translation
:------------- | :---------------------
`~> 2.0.0`     | `>= 2.0.0 and < 2.1.0`
`~> 2.1.2`     | `>= 2.1.2 and < 2.2.0`
`~> 2.1.3-dev` | `>= 2.1.3-dev and < 2.2.0`
`~> 2.0`       | `>= 2.0.0 and < 3.0.0`
`~> 2.1`       | `>= 2.1.0 and < 3.0.0`

The requirement operand after the `~>` is allowed to omit the patch version,
allowing us to express `~> 2.1` or `~> 2.1-dev`, something that wouldn't be allowed
when using the common comparison operators.

When the `:allow_pre` option is set `false` in `Version.match?/3`, the requirement
will not match a pre-release version unless the operand is a pre-release version.
The default is to always allow pre-releases but note that in
Hex `:allow_pre` is set to `false`. See the table below for examples.

Requirement    | Version     | `:allow_pre`      | Matches
:------------- | :---------- | :---------------- | :------
`~> 2.0`       | `2.1.0`     | `true` or `false` | `true`
`~> 2.0`       | `3.0.0`     | `true` or `false` | `false`
`~> 2.0.0`     | `2.0.5`     | `true` or `false` | `true`
`~> 2.0.0`     | `2.1.0`     | `true` or `false` | `false`
`~> 2.1.2`     | `2.1.6-dev` | `true`            | `true`
`~> 2.1.2`     | `2.1.6-dev` | `false`           | `false`
`~> 2.1-dev`   | `2.2.0-dev` | `true` or `false` | `true`
`~> 2.1.2-dev` | `2.1.6-dev` | `true` or `false` | `true`
`>= 2.1.0`     | `2.2.0-dev` | `true`            | `true`
`>= 2.1.0`     | `2.2.0-dev` | `false`           | `false`
`>= 2.1.0-dev` | `2.2.6-dev` | `true` or `false` | `true`

## Types

### build()

```elixir
@type build() :: String.t() | nil
```



### major()

```elixir
@type major() :: non_neg_integer()
```



### minor()

```elixir
@type minor() :: non_neg_integer()
```



### patch()

```elixir
@type patch() :: non_neg_integer()
```



### pre()

```elixir
@type pre() :: [String.t() | non_neg_integer()]
```



### requirement()

```elixir
@type requirement() :: String.t() | Version.Requirement.t()
```



### t()

```elixir
@type t() :: %Version{
  build: build(),
  major: major(),
  minor: minor(),
  patch: patch(),
  pre: pre()
}
```



### version()

```elixir
@type version() :: String.t() | t()
```



## Functions

### %Version{}
*(struct)* 


The Version struct.

It contains the fields `:major`, `:minor`, `:patch`, `:pre`, and
`:build` according to SemVer 2.0, where `:pre` is a list.

You can read those fields but you should not create a new `Version`
directly via the struct syntax. Instead use the functions in this
module.

### compare(version1, version2)

```elixir
@spec compare(version(), version()) :: :gt | :eq | :lt
```

Compares two versions.

Returns `:gt` if the first version is greater than the second one, and `:lt`
for vice versa. If the two versions are equal, `:eq` is returned.

Pre-releases are strictly less than their corresponding release versions.

Patch segments are compared lexicographically if they are alphanumeric, and
numerically otherwise.

Build segments are ignored: if two versions differ only in their build segment
they are considered to be equal.

Raises a `Version.InvalidVersionError` exception if any of the two given
versions are not parsable. If given an already parsed version this function
won't raise.

#### Examples

    iex> Version.compare("2.0.1-alpha1", "2.0.0")
    :gt
    
    iex> Version.compare("1.0.0-beta", "1.0.0-rc1")
    :lt
    
    iex> Version.compare("1.0.0-10", "1.0.0-2")
    :gt
    
    iex> Version.compare("2.0.1+build0", "2.0.1")
    :eq
    
    iex> Version.compare("invalid", "2.0.1")
    ** (Version.InvalidVersionError) invalid version: "invalid"

### compile_requirement(requirement)

```elixir
@spec compile_requirement(Version.Requirement.t()) :: Version.Requirement.t()
```

Compiles a requirement to an internal representation that may optimize matching.

The internal representation is opaque.

### match?(version, requirement, opts \\ [])

```elixir
@spec match?(version(), requirement(), keyword()) :: boolean()
```

Checks if the given version matches the specification.

Returns `true` if `version` satisfies `requirement`, `false` otherwise.
Raises a `Version.InvalidRequirementError` exception if `requirement` is not
parsable, or a `Version.InvalidVersionError` exception if `version` is not parsable.
If given an already parsed version and requirement this function won't
raise.

#### Options

- `:allow_pre` (boolean) - when `false`, pre-release versions will not match
  unless the operand is a pre-release version. Defaults to `true`.
  For examples, please refer to the table above under the "Requirements" section.

#### Examples

    iex> Version.match?("2.0.0", "> 1.0.0")
    true
    
    iex> Version.match?("2.0.0", "== 1.0.0")
    false
    
    iex> Version.match?("2.1.6-dev", "~> 2.1.2")
    true
    
    iex> Version.match?("2.1.6-dev", "~> 2.1.2", allow_pre: false)
    false
    
    iex> Version.match?("foo", "== 1.0.0")
    ** (Version.InvalidVersionError) invalid version: "foo"
    
    iex> Version.match?("2.0.0", "== == 1.0.0")
    ** (Version.InvalidRequirementError) invalid requirement: "== == 1.0.0"

### parse(string)

```elixir
@spec parse(String.t()) :: {:ok, t()} | :error
```

Parses a version string into a `Version` struct.

#### Examples

    iex> Version.parse("2.0.1-alpha1")
    {:ok, %Version{major: 2, minor: 0, patch: 1, pre: ["alpha1"]}}
    
    iex> Version.parse("2.0-alpha1")
    :error

### parse!(string)

```elixir
@spec parse!(String.t()) :: t()
```

Parses a version string into a `Version`.

If `string` is an invalid version, a `Version.InvalidVersionError` is raised.

#### Examples

    iex> Version.parse!("2.0.1-alpha1")
    %Version{major: 2, minor: 0, patch: 1, pre: ["alpha1"]}
    
    iex> Version.parse!("2.0-alpha1")
    ** (Version.InvalidVersionError) invalid version: "2.0-alpha1"

### parse_requirement(string)

```elixir
@spec parse_requirement(String.t()) :: {:ok, Version.Requirement.t()} | :error
```

Parses a version requirement string into a `Version.Requirement` struct.

#### Examples

    iex> {:ok, requirement} = Version.parse_requirement("== 2.0.1")
    iex> requirement
    Version.parse_requirement!("== 2.0.1")
    
    iex> Version.parse_requirement("== == 2.0.1")
    :error

### parse_requirement!(string)
*(since 1.8.0)* 
```elixir
@spec parse_requirement!(String.t()) :: Version.Requirement.t()
```

Parses a version requirement string into a `Version.Requirement` struct.

If `string` is an invalid requirement, a `Version.InvalidRequirementError` is raised.

#### Examples

    iex> Version.parse_requirement!("== 2.0.1")
    Version.parse_requirement!("== 2.0.1")
    
    iex> Version.parse_requirement!("== == 2.0.1")
    ** (Version.InvalidRequirementError) invalid requirement: "== == 2.0.1"

### to_string(version)
*(since 1.14.0)* 
```elixir
@spec to_string(t()) :: String.t()
```

Converts the given version to a string.

#### Examples

    iex> Version.to_string(%Version{major: 1, minor: 2, patch: 3})
    "1.2.3"
    iex> Version.to_string(Version.parse!("1.14.0-rc.0+build0"))
    "1.14.0-rc.0+build0"



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
