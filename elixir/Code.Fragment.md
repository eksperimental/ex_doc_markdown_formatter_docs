# Code.Fragment 
(Elixir v1.18.0-dev)

This module provides conveniences for analyzing fragments of
textual code and extract available information whenever possible.

This module should be considered experimental.


## Types

### position()

```elixir
@type position() :: {line :: pos_integer(), column :: pos_integer()}
```



## Functions

### container_cursor_to_quoted(fragment, opts \\ [])
*(since 1.13.0)* 
```elixir
@spec container_cursor_to_quoted(
  List.Chars.t(),
  keyword()
) ::
  {:ok, Macro.t()}
  | {:error, {location :: keyword(), binary() | {binary(), binary()}, binary()}}
```

Receives a string and returns a quoted expression
with the cursor AST position within its parent expression.

This function receives a string with an Elixir code fragment,
representing a cursor position, and converts such string to
AST with the inclusion of special `__cursor__()` node representing
the cursor position within its container (i.e. its parent).

For example, take this code, which would be given as input:

    max(some_value,

This function will return the AST equivalent to:

    max(some_value, __cursor__())

In other words, this function is capable of closing any open
brackets and insert the cursor position. Other content at the
cursor position which is not a parent is discarded.
For example, if this is given as input:

    max(some_value, another_val

It will return the same AST:

    max(some_value, __cursor__())

Similarly, if only this is given:

    max(some_va

Then it returns:

    max(__cursor__())

Calls without parenthesis are also supported, as we assume the
brackets are implicit.

Tuples, lists, maps, and binaries all retain the cursor position:

    max(some_value, [1, 2,

Returns the following AST:

    max(some_value, [1, 2, __cursor__()])

Keyword lists (and do-end blocks) are also retained. The following:

    if(some_value, do:
    if(some_value, do: :token
    if(some_value, do: 1 + val

all return:

    if(some_value, do: __cursor__())

For multi-line blocks, all previous lines are preserved.

The AST returned by this function is not safe to evaluate but
it can be analyzed and expanded.

#### Examples

Function call:

    iex> Code.Fragment.container_cursor_to_quoted("max(some_value, ")
    {:ok, {:max, [line: 1], [{:some_value, [line: 1], nil}, {:__cursor__, [line: 1], []}]}}

Containers (for example, a list):

    iex> Code.Fragment.container_cursor_to_quoted("[some, value")
    {:ok, [{:some, [line: 1], nil}, {:__cursor__, [line: 1], []}]}

If an expression is complete, then the whole expression is discarded
and only the parent is returned:

    iex> Code.Fragment.container_cursor_to_quoted("if(is_atom(var)")
    {:ok, {:if, [line: 1], [{:__cursor__, [line: 1], []}]}}

this means complete expressions themselves return only the cursor:

    iex> Code.Fragment.container_cursor_to_quoted("if(is_atom(var))")
    {:ok, {:__cursor__, [line: 1], []}}

Operators are also included from Elixir v1.15:

    iex> Code.Fragment.container_cursor_to_quoted("foo +")
    {:ok, {:+, [line: 1], [{:foo, [line: 1], nil}, {:__cursor__, [line: 1], []}]}}

In order to parse the left-side of `->` properly, which appears both
in anonymous functions and do-end blocks, the trailing fragment option
must be given with the rest of the contents:

    iex> Code.Fragment.container_cursor_to_quoted("fn x", trailing_fragment: " -> :ok end")
    {:ok, {:fn, [line: 1], [{:->, [line: 1], [[{:__cursor__, [line: 1], []}], :ok]}]}}

#### Options

- `:file` - the filename to be reported in case of parsing errors.
  Defaults to `"nofile"`.

- `:line` - the starting line of the string being parsed.
  Defaults to 1.

- `:column` - the starting column of the string being parsed.
  Defaults to 1.

- `:columns` - when `true`, attach a `:column` key to the quoted
  metadata. Defaults to `false`.

- `:token_metadata` - when `true`, includes token-related
  metadata in the expression AST, such as metadata for `do` and `end`
  tokens, for closing tokens, end of expressions, as well as delimiters
  for sigils. See `t:Macro.metadata/0`. Defaults to `false`.

- `:literal_encoder` - a function to encode literals in the AST.
  See the documentation for `Code.string_to_quoted/2` for more information.

- `:trailing_fragment` (since v1.18.0) - the rest of the contents after
  the cursor. This is necessary to correctly complete anonymous functions
  and the left-hand side of `->`


### cursor_context(fragment, opts \\ [])
*(since 1.13.0)* 
```elixir
@spec cursor_context(
  List.Chars.t(),
  keyword()
) ::
  {:alias, charlist()}
  | {:alias, inside_alias, charlist()}
  | {:dot, inside_dot, charlist()}
  | {:dot_arity, inside_dot, charlist()}
  | {:dot_call, inside_dot, charlist()}
  | :expr
  | {:local_or_var, charlist()}
  | {:local_arity, charlist()}
  | {:local_call, charlist()}
  | {:anonymous_call, inside_caller}
  | {:module_attribute, charlist()}
  | {:operator, charlist()}
  | {:operator_arity, charlist()}
  | {:operator_call, charlist()}
  | :none
  | {:sigil, charlist()}
  | {:struct, inside_struct}
  | {:unquoted_atom, charlist()}
when inside_dot:
       {:alias, charlist()}
       | {:alias, inside_alias, charlist()}
       | {:dot, inside_dot, charlist()}
       | {:module_attribute, charlist()}
       | {:unquoted_atom, charlist()}
       | {:var, charlist()}
       | :expr,
     inside_alias: {:local_or_var, charlist()} | {:module_attribute, charlist()},
     inside_struct:
       charlist()
       | {:alias, inside_alias, charlist()}
       | {:local_or_var, charlist()}
       | {:module_attribute, charlist()}
       | {:dot, inside_dot, charlist()},
     inside_caller: {:var, charlist()} | {:module_attribute, charlist()}
```

Receives a string and returns the cursor context.

This function receives a string with an Elixir code fragment,
representing a cursor position, and based on the string, it
provides contextual information about the latest token.
The return of this function can then be used to provide tips,
suggestions, and autocompletion functionality.

This function performs its analyses on tokens. This means
it does not understand how constructs are nested within each
other. See the "Limitations" section below.

Consider adding a catch-all clause when handling the return
type of this function as new cursor information may be added
in future releases.

#### Examples

    iex> Code.Fragment.cursor_context("")
    :expr
    
    iex> Code.Fragment.cursor_context("hello_wor")
    {:local_or_var, ~c"hello_wor"}

#### Return values

- `{:alias, charlist}` - the context is an alias, potentially
  a nested one, such as `Hello.Wor` or `HelloWor`

- `{:alias, inside_alias, charlist}` - the context is an alias, potentially
  a nested one, where `inside_alias` is an expression `{:module_attribute, charlist}`
  or `{:local_or_var, charlist}` and `charlist` is a static part
  Examples are `__MODULE__.Submodule` or `@hello.Submodule`

- `{:dot, inside_dot, charlist}` - the context is a dot
  where `inside_dot` is either a `{:var, charlist}`, `{:alias, charlist}`,
  `{:module_attribute, charlist}`, `{:unquoted_atom, charlist}` or a `dot`
  itself. If a var is given, this may either be a remote call or a map
  field access. Examples are `Hello.wor`, `:hello.wor`, `hello.wor`,
  `Hello.nested.wor`, `hello.nested.wor`, and `@hello.world`. If `charlist`
  is empty and `inside_dot` is an alias, then the autocompletion may either
  be an alias or a remote call.

- `{:dot_arity, inside_dot, charlist}` - the context is a dot arity
  where `inside_dot` is either a `{:var, charlist}`, `{:alias, charlist}`,
  `{:module_attribute, charlist}`, `{:unquoted_atom, charlist}` or a `dot`
  itself. If a var is given, it must be a remote arity. Examples are
  `Hello.world/`, `:hello.world/`, `hello.world/2`, and `@hello.world/2`

- `{:dot_call, inside_dot, charlist}` - the context is a dot
  call. This means parentheses or space have been added after the expression.
  where `inside_dot` is either a `{:var, charlist}`, `{:alias, charlist}`,
  `{:module_attribute, charlist}`, `{:unquoted_atom, charlist}` or a `dot`
  itself. If a var is given, it must be a remote call. Examples are
  `Hello.world(`, `:hello.world(`, ` Hello.world  `, `hello.world(`, ` hello.world  `,
  and `@hello.world(`

- `:expr` - may be any expression. Autocompletion may suggest an alias,
  local or var

- `{:local_or_var, charlist}` - the context is a variable or a local
  (import or local) call, such as `hello_wor`

- `{:local_arity, charlist}` - the context is a local (import or local)
  arity, such as `hello_world/`

- `{:local_call, charlist}` - the context is a local (import or local)
  call, such as `hello_world(` and ` hello_world  `

- `{:anonymous_call, inside_caller}` - the context is an anonymous
  call, such as `fun.(` and `@fun.(`.

- `{:module_attribute, charlist}` - the context is a module attribute,
  such as `@hello_wor`

- `{:operator, charlist}` - the context is an operator, such as `+` or
  `==`. Note textual operators, such as `when` do not appear as operators
  but rather as `:local_or_var`. `@` is never an `:operator` and always a
  `:module_attribute`

- `{:operator_arity, charlist}` - the context is an operator arity, which
  is an operator followed by /, such as `+/`, `not/` or `when/`

- `{:operator_call, charlist}` - the context is an operator call, which is
  an operator followed by space, such as ` left +  `, ` not  ` or ` x when  `

- `:none` - no context possible

- `{:sigil, charlist}` - the context is a sigil. It may be either the beginning
  of a sigil, such as `~` or `~s`, or an operator starting with `~`, such as
  `~>` and `~>>`

- `{:struct, inside_struct}` - the context is a struct, such as `%`, `%UR` or `%URI`.
  `inside_struct` can either be a `charlist` in case of a static alias or an
  expression `{:alias, inside_alias, charlist}`, `{:module_attribute, charlist}`,
  `{:local_or_var, charlist}`, `{:dot, inside_dot, charlist}`

- `{:unquoted_atom, charlist}` - the context is an unquoted atom. This
  can be any atom or an atom representing a module

We recommend looking at the test suite of this function for a complete list
of examples and their return values.

#### Limitations

The analysis is based on the current token, by analysing the last line of
the input. For example, this code:

    iex> Code.Fragment.cursor_context("%URI{")
    :expr

returns `:expr`, which suggests any variable, local function or alias
could be used. However, given we are inside a struct, the best suggestion
would be a struct field. In such cases, you can use
`container_cursor_to_quoted`, which will return the container of the AST
the cursor is currently within. You can then analyse this AST to provide
completion of field names.

As a consequence of its token-based implementation, this function considers
only the last line of the input. This means it will show suggestions inside
strings, heredocs, etc, which is intentional as it helps with doctests,
references, and more.


### surround_context(fragment, position, options \\ [])
*(since 1.13.0)* 
```elixir
@spec surround_context(List.Chars.t(), position(), keyword()) ::
  %{begin: position(), end: position(), context: context} | :none
when context:
       {:alias, charlist()}
       | {:alias, inside_alias, charlist()}
       | {:dot, inside_dot, charlist()}
       | {:local_or_var, charlist()}
       | {:local_arity, charlist()}
       | {:local_call, charlist()}
       | {:module_attribute, charlist()}
       | {:operator, charlist()}
       | {:sigil, charlist()}
       | {:struct, inside_struct}
       | {:unquoted_atom, charlist()}
       | {:keyword, charlist()}
       | {:key, charlist()}
       | {:capture_arg, charlist()},
     inside_dot:
       {:alias, charlist()}
       | {:alias, inside_alias, charlist()}
       | {:dot, inside_dot, charlist()}
       | {:module_attribute, charlist()}
       | {:unquoted_atom, charlist()}
       | {:var, charlist()}
       | :expr,
     inside_alias: {:local_or_var, charlist()} | {:module_attribute, charlist()},
     inside_struct:
       charlist()
       | {:alias, inside_alias, charlist()}
       | {:local_or_var, charlist()}
       | {:module_attribute, charlist()}
       | {:dot, inside_dot, charlist()}
```

Receives a string and returns the surround context.

This function receives a string with an Elixir code fragment
and a `position`. It returns a map containing the beginning
and ending of the identifier alongside its context, or `:none`
if there is nothing with a known context. This is useful to
provide mouse-over and highlight functionality in editors.

The difference between `cursor_context/2` and `surround_context/3`
is that the former assumes the expression in the code fragment
is incomplete. For example, `do` in `cursor_context/2` may be
a keyword or a variable or a local call, while `surround_context/3`
assumes the expression in the code fragment is complete, therefore
`do` would always be a keyword.

The `position` contains both the `line` and `column`, both starting
with the index of 1. The column must precede the surrounding expression.
For example, the expression `foo`, will return something for the columns
1, 2, and 3, but not 4:

    foo
    ^ column 1
    
    foo
     ^ column 2
    
    foo
      ^ column 3
    
    foo
       ^ column 4

The returned map contains the column the expression starts and the
first column after the expression ends.

Similar to `cursor_context/2`, this function is also token-based
and may not be accurate under all circumstances. See the
"Return values" and "Limitations" section under `cursor_context/2`
for more information.

#### Examples

    iex> Code.Fragment.surround_context("foo", {1, 1})
    %{begin: {1, 1}, context: {:local_or_var, ~c"foo"}, end: {1, 4}}

#### Differences to `cursor_context/2`

Because `surround_context/3` attempts to capture complex expressions,
it has some differences to `cursor_context/2`:

- `dot_call`/`dot_arity` and `operator_call`/`operator_arity`
  are collapsed into `dot` and `operator` contexts respectively
  as there aren't any meaningful distinctions between them

- On the other hand, this function still makes a distinction between
  `local_call`/`local_arity` and `local_or_var`, since the latter can
  be a local or variable

- `@` when not followed by any identifier is returned as `{:operator, ~c"@"}`
  (in contrast to `{:module_attribute, ~c""}` in `cursor_context/2`

- This function never returns empty sigils `{:sigil, ~c""}` or empty structs
  `{:struct, ~c""}` as context

- This function returns keywords as `{:keyword, ~c"do"}`

- This function never returns `:expr`

We recommend looking at the test suite of this function for a complete list
of examples and their return values.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
