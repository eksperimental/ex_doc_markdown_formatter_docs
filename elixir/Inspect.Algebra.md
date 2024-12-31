# Inspect.Algebra 
(Elixir v1.18.0-dev)

A set of functions for creating and manipulating algebra
documents.

This module implements the functionality described in
["Strictly Pretty" (2000) by Christian Lindig](https://lindig.github.io/papers/strictly-pretty-2000.pdf) with small
additions, like support for binary nodes and a break mode that
maximises use of horizontal space.

    iex> Inspect.Algebra.empty()
    :doc_nil
    
    iex> "foo"
    "foo"

With the functions in this module, we can concatenate different
elements together and render them:

    iex> doc = Inspect.Algebra.concat(Inspect.Algebra.empty(), "foo")
    iex> Inspect.Algebra.format(doc, 80)
    ["foo"]

The functions `nest/2`, `space/2` and `line/2` help you put the
document together into a rigid structure. However, the document
algebra gets interesting when using functions like `glue/3` and
`group/1`. A glue inserts a break between two documents. A group
indicates a document that must fit the current line, otherwise
breaks are rendered as new lines. Let's glue two docs together
with a break, group it and then render it:

    iex> doc = Inspect.Algebra.glue("a", " ", "b")
    iex> doc = Inspect.Algebra.group(doc)
    iex> Inspect.Algebra.format(doc, 80)
    ["a", " ", "b"]

Note that the break was represented as is, because we haven't reached
a line limit. Once we do, it is replaced by a newline:

    iex> doc = Inspect.Algebra.glue(String.duplicate("a", 20), " ", "b")
    iex> doc = Inspect.Algebra.group(doc)
    iex> Inspect.Algebra.format(doc, 10)
    ["aaaaaaaaaaaaaaaaaaaa", "\n", "b"]

This module uses the byte size to compute how much space there is
left. If your document contains strings, then those need to be
wrapped in `string/1`, which then relies on `String.length/1` to
precompute the document size.

Finally, this module also contains Elixir related functions, a bit
tied to Elixir formatting, such as `to_doc/2`.

## Implementation details

The implementation of `Inspect.Algebra` is based on the Strictly Pretty
paper by [Lindig](https://lindig.github.io/papers/strictly-pretty-2000.pdf) which builds on top of previous pretty printing
algorithms but is tailored to strict languages, such as Elixir.
The core idea in the paper is the use of explicit document groups which
are rendered as flat (breaks as spaces) or as break (breaks as newlines).

This implementation provides two types of breaks: `:strict` and `:flex`.
When a group does not fit, all strict breaks are treated as newlines.
Flex breaks, however, are re-evaluated on every occurrence and may still
be rendered flat. See `break/1` and `flex_break/1` for more information.

This implementation also adds `force_unfit/1` and `next_break_fits/2` which
give more control over the document fitting.


## Guards

### is_doc(doc)
*(macro)* 




## Types

### t()

```elixir
@type t() ::
  binary()
  | :doc_line
  | :doc_nil
  | doc_break()
  | doc_collapse()
  | doc_color()
  | doc_cons()
  | doc_fits()
  | doc_force()
  | doc_group()
  | doc_nest()
  | doc_string()
  | doc_limit()
```



## Functions

### break(string \\ &quot; &quot;)

```elixir
@spec break(binary()) :: doc_break()
```

Returns a break document based on the given `string`.

This break can be rendered as a linebreak or as the given `string`,
depending on the `mode` of the chosen layout.

#### Examples

Let's create a document by concatenating two strings with a break between
them:

    iex> doc = Inspect.Algebra.concat(["a", Inspect.Algebra.break("\t"), "b"])
    iex> Inspect.Algebra.format(doc, 80)
    ["a", "\t", "b"]

Note that the break was represented with the given string, because we didn't
reach a line limit. Once we do, it is replaced by a newline:

    iex> break = Inspect.Algebra.break("\t")
    iex> doc = Inspect.Algebra.concat([String.duplicate("a", 20), break, "b"])
    iex> doc = Inspect.Algebra.group(doc)
    iex> Inspect.Algebra.format(doc, 10)
    ["aaaaaaaaaaaaaaaaaaaa", "\n", "b"]


### collapse_lines(max)
*(since 1.6.0)* 
```elixir
@spec collapse_lines(pos_integer()) :: doc_collapse()
```

Collapse any new lines and whitespace following this
node, emitting up to `max` new lines.


### color(doc, color)
*(since 1.18.0)* 
```elixir
@spec color(t(), binary()) :: t()
```

Colors a document with the given color (preceding the document itself).


### color(doc, key, opts)


This function is deprecated. Use color_doc/3 instead.


### color_doc(doc, color_key, opts)
*(since 1.18.0)* 
```elixir
@spec color_doc(t(), Inspect.Opts.color_key(), Inspect.Opts.t()) :: t()
```

Colors a document if the `color_key` has a color in the options.


### concat(docs)

```elixir
@spec concat([t()]) :: t()
```

Concatenates a list of documents returning a new document.

#### Examples

    iex> doc = Inspect.Algebra.concat(["a", "b", "c"])
    iex> Inspect.Algebra.format(doc, 80)
    ["a", "b", "c"]


### concat(doc1, doc2)

```elixir
@spec concat(t(), t()) :: t()
```

Concatenates two document entities returning a new document.

#### Examples

    iex> doc = Inspect.Algebra.concat("hello", "world")
    iex> Inspect.Algebra.format(doc, 80)
    ["hello", "world"]


### container_doc(left, collection, right, inspect_opts, fun, opts \\ [])
*(since 1.6.0)* 
```elixir
@spec container_doc(
  t(),
  [term()],
  t(),
  Inspect.Opts.t(),
  (term(), Inspect.Opts.t() -&gt; t()),
  keyword()
) ::
  t()
```

Wraps `collection` in `left` and `right` according to limit and contents.

It uses the given `left` and `right` documents as surrounding and the
separator document `separator` to separate items in `docs`. If all entries
in the collection are simple documents (texts or strings), then this function
attempts to put as much as possible on the same line. If they are not simple,
only one entry is shown per line if they do not fit.

The limit in the given `inspect_opts` is respected and when reached this
function stops processing and outputs `"..."` instead.

#### Options

- `:separator` - the separator used between each doc
- `:break` - If `:strict`, always break between each element. If `:flex`,
  breaks only when necessary. If `:maybe`, chooses `:flex` only if all
  elements are text-based, otherwise is `:strict`

#### Examples

    iex> inspect_opts = %Inspect.Opts{limit: :infinity}
    iex> fun = fn i, _opts -> to_string(i) end
    iex> doc = Inspect.Algebra.container_doc("[", Enum.to_list(1..5), "]", inspect_opts, fun)
    iex> Inspect.Algebra.format(doc, 5) |> IO.iodata_to_binary()
    "[1,\n 2,\n 3,\n 4,\n 5]"
    
    iex> inspect_opts = %Inspect.Opts{limit: 3}
    iex> fun = fn i, _opts -> to_string(i) end
    iex> doc = Inspect.Algebra.container_doc("[", Enum.to_list(1..5), "]", inspect_opts, fun)
    iex> Inspect.Algebra.format(doc, 20) |> IO.iodata_to_binary()
    "[1, 2, 3, ...]"
    
    iex> inspect_opts = %Inspect.Opts{limit: 3}
    iex> fun = fn i, _opts -> to_string(i) end
    iex> opts = [separator: "!"]
    iex> doc = Inspect.Algebra.container_doc("[", Enum.to_list(1..5), "]", inspect_opts, fun, opts)
    iex> Inspect.Algebra.format(doc, 20) |> IO.iodata_to_binary()
    "[1! 2! 3! ...]"


### empty()

```elixir
@spec empty() :: :doc_nil
```

Returns a document entity used to represent nothingness.

#### Examples

    iex> Inspect.Algebra.empty()
    :doc_nil


### flex_break(string \\ &quot; &quot;)
*(since 1.6.0)* 
```elixir
@spec flex_break(binary()) :: doc_break()
```

Returns a flex break document based on the given `string`.

A flex break still causes a group to break, like `break/1`,
but it is re-evaluated when the documented is rendered.

For example, take a group document represented as `[1, 2, 3]`
where the space after every comma is a break. When the document
above does not fit a single line, all breaks are enabled,
causing the document to be rendered as:

    [1,
     2,
     3]

However, if flex breaks are used, then each break is re-evaluated
when rendered, so the document could be possible rendered as:

    [1, 2,
     3]

Hence the name "flex". they are more flexible when it comes
to the document fitting. On the other hand, they are more expensive
since each break needs to be re-evaluated.

This function is used by `container_doc/6` and friends to the
maximum number of entries on the same line.


### flex_glue(doc1, break_string \\ &quot; &quot;, doc2)
*(since 1.6.0)* 
```elixir
@spec flex_glue(t(), binary(), t()) :: t()
```

Glues two documents (`doc1` and `doc2`) inserting a
`flex_break/1` given by `break_string` between them.

This function is used by `container_doc/6` and friends
to the maximum number of entries on the same line.


### fold(docs, folder_fun)
*(since 1.18.0)* 
```elixir
@spec fold([t()], (t(), t() -&gt; t())) :: t()
```

Folds a list of documents into a document using the given folder function.

The list of documents is folded "from the right"; in that, this function is
similar to `List.foldr/3`, except that it doesn't expect an initial
accumulator and uses the last element of `docs` as the initial accumulator.

#### Examples

    iex> docs = ["A", "B", "C"]
    iex> docs =
    ...>   Inspect.Algebra.fold(docs, fn doc, acc ->
    ...>     Inspect.Algebra.concat([doc, "!", acc])
    ...>   end)
    iex> Inspect.Algebra.format(docs, 80)
    ["A", "!", "B", "!", "C"]


### fold_doc(docs, folder_fun)


This function is deprecated. Use fold/2 instead.


### force_unfit(doc)
*(since 1.6.0)* 
```elixir
@spec force_unfit(t()) :: doc_force()
```

Forces the current group to be unfit.


### format(doc, width)

```elixir
@spec format(t(), non_neg_integer() | :infinity) :: iodata()
```

Formats a given document for a given width.

Takes the maximum width and a document to print as its arguments
and returns an IO data representation of the best layout for the
document to fit in the given width.

The document starts flat (without breaks) until a group is found.

#### Examples

    iex> doc = Inspect.Algebra.glue("hello", " ", "world")
    iex> doc = Inspect.Algebra.group(doc)
    iex> doc |> Inspect.Algebra.format(30) |> IO.iodata_to_binary()
    "hello world"
    iex> doc |> Inspect.Algebra.format(10) |> IO.iodata_to_binary()
    "hello\nworld"


### glue(doc1, break_string \\ &quot; &quot;, doc2)

```elixir
@spec glue(t(), binary(), t()) :: t()
```

Glues two documents (`doc1` and `doc2`) inserting the given
break `break_string` between them.

For more information on how the break is inserted, see `break/1`.

#### Examples

    iex> doc = Inspect.Algebra.glue("hello", "world")
    iex> Inspect.Algebra.format(doc, 80)
    ["hello", " ", "world"]
    
    iex> doc = Inspect.Algebra.glue("hello", "\t", "world")
    iex> Inspect.Algebra.format(doc, 80)
    ["hello", "\t", "world"]


### group(doc, mode \\ :self)

```elixir
@spec group(t(), :self | :inherit) :: doc_group()
```

Returns a group containing the specified document `doc`.

Documents in a group are attempted to be rendered together
to the best of the renderer ability.

The group mode can also be set to `:inherit`, which means it
automatically breaks if the parent group has broken too.

#### Examples

    iex> doc =
    ...>   Inspect.Algebra.group(
    ...>     Inspect.Algebra.concat(
    ...>       Inspect.Algebra.group(
    ...>         Inspect.Algebra.concat(
    ...>           "Hello,",
    ...>           Inspect.Algebra.concat(
    ...>             Inspect.Algebra.break(),
    ...>             "A"
    ...>           )
    ...>         )
    ...>       ),
    ...>       Inspect.Algebra.concat(
    ...>         Inspect.Algebra.break(),
    ...>         "B"
    ...>       )
    ...>     )
    ...>   )
    iex> Inspect.Algebra.format(doc, 80)
    ["Hello,", " ", "A", " ", "B"]
    iex> Inspect.Algebra.format(doc, 6)
    ["Hello,", "\n", "A", "\n", "B"]


### line()
*(since 1.6.0)* 
```elixir
@spec line() :: t()
```

A mandatory linebreak.

A group with linebreaks will fit if all lines in the group fit.

#### Examples

    iex> doc =
    ...>   Inspect.Algebra.concat(
    ...>     Inspect.Algebra.concat(
    ...>       "Hughes",
    ...>       Inspect.Algebra.line()
    ...>     ),
    ...>     "Wadler"
    ...>   )
    iex> Inspect.Algebra.format(doc, 80)
    ["Hughes", "\n", "Wadler"]


### line(doc1, doc2)

```elixir
@spec line(t(), t()) :: t()
```

Inserts a mandatory linebreak between two documents.

See `line/0`.

#### Examples

    iex> doc = Inspect.Algebra.line("Hughes", "Wadler")
    iex> Inspect.Algebra.format(doc, 80)
    ["Hughes", "\n", "Wadler"]


### nest(doc, level, mode \\ :always)

```elixir
@spec nest(t(), non_neg_integer() | :cursor | :reset, :always | :break) ::
  doc_nest() | t()
```

Nests the given document at the given `level`.

If `level` is an integer, that's the indentation appended
to line breaks whenever they occur. If the level is `:cursor`,
the current position of the "cursor" in the document becomes
the nesting. If the level is `:reset`, it is set back to 0.

`mode` can be `:always`, which means nesting always happen,
or `:break`, which means nesting only happens inside a group
that has been broken.

#### Examples

    iex> doc = Inspect.Algebra.nest(Inspect.Algebra.glue("hello", "world"), 5)
    iex> doc = Inspect.Algebra.group(doc)
    iex> Inspect.Algebra.format(doc, 5)
    ["hello", "\n     ", "world"]


### next_break_fits(doc, mode \\ :enabled)
*(since 1.6.0)* 
```elixir
@spec next_break_fits(t(), :enabled | :disabled) :: doc_fits()
```

Considers the next break as fit.

`mode` can be `:enabled` or `:disabled`. When `:enabled`,
it will consider the document as fit as soon as it finds
the next break, effectively cancelling the break. It will
also ignore any `force_unfit/1` in search of the next break.

When disabled, it behaves as usual and it will ignore
any further `next_break_fits/2` instruction.

#### Examples

This is used by Elixir's code formatter to avoid breaking
code at some specific locations. For example, consider this
code:

    some_function_call(%{..., key: value, ...})

Now imagine that this code does not fit its line. The code
formatter introduces breaks inside `(` and `)` and inside
`%{` and `}`. Therefore the document would break as:

    some_function_call(
      %{
        ...,
        key: value,
        ...
      }
    )

The formatter wraps the algebra document representing the
map in `next_break_fits/1` so the code is formatted as:

    some_function_call(%{
      ...,
      key: value,
      ...
    })


### no_limit(doc)
*(since 1.14.0)* 
```elixir
@spec no_limit(t()) :: t()
```

Disable any rendering limit while rendering the given document.

#### Examples

    iex> doc = Inspect.Algebra.glue("hello", "world") |> Inspect.Algebra.group()
    iex> Inspect.Algebra.format(doc, 10)
    ["hello", "\n", "world"]
    iex> doc = Inspect.Algebra.no_limit(doc)
    iex> Inspect.Algebra.format(doc, 10)
    ["hello", " ", "world"]


### space(doc1, doc2)

```elixir
@spec space(t(), t()) :: t()
```

Inserts a mandatory single space between two documents.

#### Examples

    iex> doc = Inspect.Algebra.space("Hughes", "Wadler")
    iex> Inspect.Algebra.format(doc, 5)
    ["Hughes", " ", "Wadler"]


### string(string)
*(since 1.6.0)* 
```elixir
@spec string(String.t()) :: doc_string()
```

Creates a document represented by string.

While `Inspect.Algebra` accepts binaries as documents,
those are counted by binary size. On the other hand,
`string` documents are measured in terms of graphemes
towards the document size.

#### Examples

The following document has 10 bytes and therefore it
does not format to width 9 without breaks:

    iex> doc = Inspect.Algebra.glue("olá", " ", "mundo")
    iex> doc = Inspect.Algebra.group(doc)
    iex> Inspect.Algebra.format(doc, 9)
    ["olá", "\n", "mundo"]

However, if we use `string`, then the string length is
used, instead of byte size, correctly fitting:

    iex> string = Inspect.Algebra.string("olá")
    iex> doc = Inspect.Algebra.glue(string, " ", "mundo")
    iex> doc = Inspect.Algebra.group(doc)
    iex> Inspect.Algebra.format(doc, 9)
    ["olá", " ", "mundo"]


### to_doc(term, opts)

```elixir
@spec to_doc(any(), Inspect.Opts.t()) :: t()
```

Converts an Elixir term to an algebra document
according to the `Inspect` protocol.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
