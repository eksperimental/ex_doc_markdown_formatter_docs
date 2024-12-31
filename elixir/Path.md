# Path 
(Elixir v1.18.0-dev)

This module provides conveniences for manipulating or
retrieving file system paths.

The functions in this module may receive chardata as
arguments and will always return a string encoded in UTF-8. Chardata
is a string or a list of characters and strings, see `t:IO.chardata/0`.
If a binary is given, in whatever encoding, its encoding will be kept.

The majority of the functions in this module do not
interact with the file system, except for a few functions
that require it (like `wildcard/2` and `expand/1`).

## Types

### t()

```elixir
@type t() :: IO.chardata()
```

A path.

## Functions

### absname(path)

```elixir
@spec absname(t()) :: binary()
```

Converts the given path to an absolute one.

Unlike `expand/1`, no attempt is made to resolve `..`, `.`, or `~`.

#### Examples

##### Unix-like operating systems

    Path.absname("foo")
    #=> "/usr/local/foo"
    
    Path.absname("../x")
    #=> "/usr/local/../x"

##### Windows

    Path.absname("foo")
    #=> "D:/usr/local/foo"
    
    Path.absname("../x")
    #=> "D:/usr/local/../x"

### absname(path, relative_to)

```elixir
@spec absname(t(), t() | (-&gt; t())) :: binary()
```

Builds a path from `relative_to` to `path`.

If `path` is already an absolute path, `relative_to` is ignored. See also
`relative_to/3`. `relative_to` is either a path or an anonymous function,
which is invoked only when necessary, that returns a path.

Unlike `expand/2`, no attempt is made to resolve `..`, `.` or `~`.

#### Examples

    iex> Path.absname("foo", "bar")
    "bar/foo"
    
    iex> Path.absname("../x", "bar")
    "bar/../x"
    
    iex> Path.absname("foo", fn -> "lazy" end)
    "lazy/foo"

### basename(path)

```elixir
@spec basename(t()) :: binary()
```

Returns the last component of the path or the path
itself if it does not contain any directory separators.

#### Examples

    iex> Path.basename("foo")
    "foo"
    
    iex> Path.basename("foo/bar")
    "bar"
    
    iex> Path.basename("lib/module/submodule.ex")
    "submodule.ex"
    
    iex> Path.basename("/")
    ""

### basename(path, extension)

```elixir
@spec basename(t(), t()) :: binary()
```

Returns the last component of `path` with the `extension`
stripped.

This function should be used to remove a specific
extension which may or may not be there.

#### Examples

    iex> Path.basename("~/foo/bar.ex", ".ex")
    "bar"
    
    iex> Path.basename("~/foo/bar.exs", ".ex")
    "bar.exs"
    
    iex> Path.basename("~/foo/bar.old.ex", ".ex")
    "bar.old"

### dirname(path)

```elixir
@spec dirname(t()) :: binary()
```

Returns the directory component of `path`.

#### Examples

    iex> Path.dirname("/foo/bar.ex")
    "/foo"
    
    iex> Path.dirname("/foo/bar/baz.ex")
    "/foo/bar"
    
    iex> Path.dirname("/foo/bar/")
    "/foo/bar"
    
    iex> Path.dirname("bar.ex")
    "."

### expand(path)

```elixir
@spec expand(t()) :: binary()
```

Converts the path to an absolute one, expanding
any `.` and `..` components and a leading `~`.

If a relative path is provided it is expanded relatively to
the current working directory.

#### Examples

    Path.expand("/foo/bar/../baz")
    #=> "/foo/baz"
    
    Path.expand("foo/bar/../baz")
    #=> "$PWD/foo/baz"

### expand(path, relative_to)

```elixir
@spec expand(t(), t()) :: binary()
```

Expands the path relative to the path given as the second argument
expanding any `.` and `..` characters.

If the path is already an absolute path, `relative_to` is ignored.

Note that this function treats a `path` with a leading `~` as
an absolute one.

The second argument is first expanded to an absolute path.

#### Examples

    # Assuming that the absolute path to baz is /quux/baz
    Path.expand("foo/bar/../bar", "baz")
    #=> "/quux/baz/foo/bar"
    
    Path.expand("foo/bar/../bar", "/baz")
    #=> "/baz/foo/bar"
    
    Path.expand("/foo/bar/../bar", "/baz")
    #=> "/foo/bar"

### extname(path)

```elixir
@spec extname(t()) :: binary()
```

Returns the extension of the last component of `path`.

For filenames starting with a dot and without an extension, it returns
an empty string.

See `basename/1` and `rootname/1` for related functions to extract
information from paths.

#### Examples

    iex> Path.extname("foo.erl")
    ".erl"
    
    iex> Path.extname("~/foo/bar")
    ""
    
    iex> Path.extname(".gitignore")
    ""

### join(list)

```elixir
@spec join([t(), ...]) :: binary()
```

Joins a list of paths.

This function should be used to convert a list of paths to a path.
Note that any trailing slash is removed when joining.

Raises an error if the given list of paths is empty.

#### Examples

    iex> Path.join(["~", "foo"])
    "~/foo"
    
    iex> Path.join(["foo"])
    "foo"
    
    iex> Path.join(["/", "foo", "bar/"])
    "/foo/bar"

### join(left, right)

```elixir
@spec join(t(), t()) :: binary()
```

Joins two paths.

The right path will always be expanded to its relative format
and any trailing slash will be removed when joining.

#### Examples

    iex> Path.join("foo", "bar")
    "foo/bar"
    
    iex> Path.join("/foo", "/bar/")
    "/foo/bar"

The functions in this module support chardata, so giving a list will
treat it as a single entity:

    iex> Path.join("foo", ["bar", "fiz"])
    "foo/barfiz"
    
    iex> Path.join(["foo", "bar"], "fiz")
    "foobar/fiz"

Use `join/1` if you need to join a list of paths instead.

### relative(name)

```elixir
@spec relative(t()) :: binary()
```

Forces the path to be a relative path.

If an absolute path is given, it is stripped from its root component.

#### Examples

##### Unix-like operating systems

    Path.relative("/usr/local/bin")   #=> "usr/local/bin"
    Path.relative("usr/local/bin")    #=> "usr/local/bin"
    Path.relative("../usr/local/bin") #=> "../usr/local/bin"

##### Windows

    Path.relative("D:/usr/local/bin") #=> "usr/local/bin"
    Path.relative("usr/local/bin")    #=> "usr/local/bin"
    Path.relative("D:bar.ex")         #=> "bar.ex"
    Path.relative("/bar/foo.ex")      #=> "bar/foo.ex"

### relative_to(path, cwd, opts \\ [])

```elixir
@spec relative_to(t(), t(), keyword()) :: binary()
```

Returns the direct relative path from `path` in relation to `cwd`.

In other words, this function attempts to return a path such that
`Path.expand(result, cwd)` points to `path`. This function aims
to return a relative path whenever possible, but that's not guaranteed:

- If both paths are relative, a relative path is always returned

- If both paths are absolute, a relative path may be returned if
  they share a common prefix. You can pass the `:force` option to
  force this function to traverse up, but even then a relative
  path is not guaranteed (for example, if the absolute paths
  belong to different drives on Windows)

- If a mixture of paths are given, the result will always match
  the given `path` (the first argument)

This function expands `.` and `..` entries without traversing the
file system, so it assumes no symlinks between the paths. See
`safe_relative_to/2` for a safer alternative.

#### Options

- `:force` - (boolean since v1.16.0) if `true` forces a relative
  path to be returned by traversing the path up. Except if the paths
  are in different volumes on Windows. Defaults to `false`.

#### Examples

##### With relative `cwd`

If both paths are relative, a minimum path is computed:

    Path.relative_to("tmp/foo/bar", "tmp")      #=> "foo/bar"
    Path.relative_to("tmp/foo/bar", "tmp/foo")  #=> "bar"
    Path.relative_to("tmp/foo/bar", "tmp/bat")  #=> "../foo/bar"

If an absolute path is given with relative `cwd`, it is returned as:

    Path.relative_to("/usr/foo/bar", "tmp/bat")  #=> "/usr/foo/bar"

##### With absolute `cwd`

If both paths are absolute, a relative is computed if possible,
without traversing up:

    Path.relative_to("/usr/local/foo", "/usr/local")      #=> "foo"
    Path.relative_to("/usr/local/foo", "/")               #=> "usr/local/foo"
    Path.relative_to("/usr/local/foo", "/etc")            #=> "/usr/local/foo"
    Path.relative_to("/usr/local/foo", "/usr/local/foo")  #=> "."
    Path.relative_to("/usr/local/../foo", "/usr/foo")     #=> "."
    Path.relative_to("/usr/local/../foo/bar", "/usr/foo") #=> "bar"

If `:force` is set to `true` paths are traversed up:

    Path.relative_to("/usr", "/usr/local", force: true)          #=> ".."
    Path.relative_to("/usr/foo", "/usr/local", force: true)      #=> "../foo"
    Path.relative_to("/usr/../foo/bar", "/etc/foo", force: true) #=> "../../foo/bar"

If a relative path is given, it is assumed to be relative to the
given path, so the path is returned with "." and ".." expanded:

    Path.relative_to(".", "/usr/local")          #=> "."
    Path.relative_to("foo", "/usr/local")        #=> "foo"
    Path.relative_to("foo/../bar", "/usr/local") #=> "bar"
    Path.relative_to("foo/..", "/usr/local")     #=> "."
    Path.relative_to("../foo", "/usr/local")     #=> "../foo"

### relative_to_cwd(path, opts \\ [])

```elixir
@spec relative_to_cwd(
  t(),
  keyword()
) :: binary()
```

Convenience to get the path relative to the current working
directory.

If, for some reason, the current working directory
cannot be retrieved, this function returns the given `path`.

Check `relative_to/3` for the supported options.

### rootname(path)

```elixir
@spec rootname(t()) :: binary()
```

Returns the `path` with the `extension` stripped.

#### Examples

    iex> Path.rootname("/foo/bar")
    "/foo/bar"
    
    iex> Path.rootname("/foo/bar.ex")
    "/foo/bar"

### rootname(path, extension)

```elixir
@spec rootname(t(), t()) :: binary()
```

Returns the `path` with the `extension` stripped.

This function should be used to remove a specific extension which may
or may not be there.

#### Examples

    iex> Path.rootname("/foo/bar.erl", ".erl")
    "/foo/bar"
    
    iex> Path.rootname("/foo/bar.erl", ".ex")
    "/foo/bar.erl"

### safe_relative(path, cwd \\ File.cwd!())
*(since 1.14.0)* 
```elixir
@spec safe_relative(t(), t()) :: {:ok, binary()} | :error
```

Returns a relative path that is protected from directory-traversal attacks.

The given relative path is sanitized by eliminating `..` and `.` components.

This function checks that, after expanding those components, the path is still "safe".
Paths are considered unsafe if either of these is true:

- The path is not relative, such as `"/foo/bar"`.

- A `..` component would make it so that the path would traverse up above
  the root of `relative_to`.

- A symbolic link in the path points to something above the root of `cwd`.

#### Examples

    iex> Path.safe_relative("foo")
    {:ok, "foo"}
    
    iex> Path.safe_relative("deps/my_dep/app.beam")
    {:ok, "deps/my_dep/app.beam"}
    
    iex> Path.safe_relative("deps/my_dep/./build/../app.beam", File.cwd!())
    {:ok, "deps/my_dep/app.beam"}
    
    iex> Path.safe_relative("my_dep/../..")
    :error
    
    iex> Path.safe_relative("/usr/local", File.cwd!())
    :error

### safe_relative_to(path, cwd)
*(since 1.14.0)* 
```elixir
@spec safe_relative_to(t(), t()) :: {:ok, binary()} | :error
```
**This function is deprecated. Use safe_relative/2 instead.**

Returns a relative path that is protected from directory-traversal attacks.

See `safe_relative/2` for a non-deprecated version of this API.

### split(path)

```elixir
@spec split(t()) :: [binary()]
```

Splits the path into a list at the path separator.

If an empty string is given, returns an empty list.

On Windows, path is split on both `"\"` and `"/"` separators
and the driver letter, if there is one, is always returned
in lowercase.

#### Examples

    iex> Path.split("")
    []
    
    iex> Path.split("foo")
    ["foo"]
    
    iex> Path.split("/foo/bar")
    ["/", "foo", "bar"]

### type(name)

```elixir
@spec type(t()) :: :absolute | :relative | :volumerelative
```

Returns the path type.

#### Examples

##### Unix-like operating systems

    Path.type("/")                #=> :absolute
    Path.type("/usr/local/bin")   #=> :absolute
    Path.type("usr/local/bin")    #=> :relative
    Path.type("../usr/local/bin") #=> :relative
    Path.type("~/file")           #=> :relative

##### Windows

    Path.type("D:/usr/local/bin") #=> :absolute
    Path.type("usr/local/bin")    #=> :relative
    Path.type("D:bar.ex")         #=> :volumerelative
    Path.type("/bar/foo.ex")      #=> :volumerelative

### wildcard(glob, opts \\ [])

```elixir
@spec wildcard(
  t(),
  keyword()
) :: [binary()]
```

Traverses paths according to the given `glob` expression and returns a
list of matches.

The wildcard looks like an ordinary path, except that the following
"wildcard characters" are interpreted in a special way:

- `?` - matches one character.

- `*` - matches any number of characters up to the end of the filename, the
  next dot, or the next slash.

- `**` - two adjacent `*`'s used as a single pattern will match all
  files and zero or more directories and subdirectories.

- `[char1,char2,...]` - matches any of the characters listed; two
  characters separated by a hyphen will match a range of characters.
  Do not add spaces before and after the comma as it would then match
  paths containing the space character itself.

- `{item1,item2,...}` - matches one of the alternatives.
  Do not add spaces before and after the comma as it would then match
  paths containing the space character itself.

Other characters represent themselves. Only paths that have
exactly the same character in the same position will match. Note
that matching is case-sensitive: `"a"` will not match `"A"`.

Directory separators must always be written as `/`, even on Windows.
You may call `Path.expand/1` to normalize the path before invoking
this function.

A character preceded by `\\` loses its special meaning.
Note that `\\` must be written as `\\\\` in a string literal.
For example, `"\\\\?*"` will match any filename starting with `?.`.

By default, the patterns `*` and `?` do not match files starting
with a dot `.`. See the `:match_dot` option in the "Options" section
below.

#### Options

- `:match_dot` - (boolean) if `false`, the special wildcard characters `*` and `?`
  will not match files starting with a dot (`.`). If `true`, files starting with
  a `.` will not be treated specially. Defaults to `false`.

#### Examples

Imagine you have a directory called `projects` with three Elixir projects
inside of it: `elixir`, `ex_doc`, and `plug`. You can find all `.beam` files
inside the `ebin` directory of each project as follows:

    Path.wildcard("projects/*/ebin/**/*.beam")

If you want to search for both `.beam` and `.app` files, you could do:

    Path.wildcard("projects/*/ebin/**/*.{beam,app}")



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
