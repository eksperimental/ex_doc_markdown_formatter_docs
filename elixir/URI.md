# URI 
(Elixir v1.18.0-dev)

Utilities for working with URIs.

This module provides functions for working with URIs (for example, parsing
URIs or encoding query strings). The functions in this module are implemented
according to [RFC 3986](https://tools.ietf.org/html/rfc3986).

Additionally, the Erlang [`:uri_string` module](\`:uri_string\`) provides certain functionalities,
such as RFC 3986 compliant URI normalization.

## Types

### authority()

```elixir
@opaque authority()
```
**This opaque is deprecated. The authority field is deprecated.**



### t()

```elixir
@type t() :: %URI{
  authority: authority(),
  fragment: nil | binary(),
  host: nil | binary(),
  path: nil | binary(),
  port: nil | :inet.port_number(),
  query: nil | binary(),
  scheme: nil | binary(),
  userinfo: nil | binary()
}
```



## Functions

### %URI{}
*(struct)* 


The URI struct.

The fields are defined to match the following URI representation
(with field names between brackets):

    [scheme]://[userinfo]@[host]:[port][path]?[query]#[fragment]

Note the `authority` field is deprecated. `parse/1` will still
populate it for backwards compatibility but you should generally
avoid setting or getting it.

### append_path(uri, path)
*(since 1.15.0)* 
```elixir
@spec append_path(t(), String.t()) :: t()
```

Appends `path` to the given `uri`.

Path must start with `/` and cannot contain additional URL components like
fragments or query strings. This function further assumes the path is valid and
it does not contain a query string or fragment parts.

#### Examples

    iex> URI.append_path(URI.parse("http://example.com/foo/?x=1"), "/my-path") |> URI.to_string()
    "http://example.com/foo/my-path?x=1"
    
    iex> URI.append_path(URI.parse("http://example.com"), "my-path")
    ** (ArgumentError) path must start with "/", got: "my-path"

### append_query(uri, query)
*(since 1.14.0)* 
```elixir
@spec append_query(t(), binary()) :: t()
```

Appends `query` to the given `uri`.

The given `query` is not automatically encoded, use `encode/2` or `encode_www_form/1`.

#### Examples

    iex> URI.append_query(URI.parse("http://example.com/"), "x=1") |> URI.to_string()
    "http://example.com/?x=1"
    
    iex> URI.append_query(URI.parse("http://example.com/?x=1"), "y=2") |> URI.to_string()
    "http://example.com/?x=1&y=2"
    
    iex> URI.append_query(URI.parse("http://example.com/?x=1"), "x=2") |> URI.to_string()
    "http://example.com/?x=1&x=2"

### char_reserved?(character)

```elixir
@spec char_reserved?(byte()) :: boolean()
```

Checks if `character` is a reserved one in a URI.

As specified in [RFC 3986, section 2.2](https://tools.ietf.org/html/rfc3986#section-2.2),
the following characters are reserved: `:`, `/`, `?`, `#`, `[`, `]`, `@`, `!`, `$`, `&`, `'`, `(`, `)`, `*`, `+`, `,`, `;`, `=`

#### Examples

    iex> URI.char_reserved?(?+)
    true

### char_unescaped?(character)

```elixir
@spec char_unescaped?(byte()) :: boolean()
```

Checks if `character` is allowed unescaped in a URI.

This is the default used by `URI.encode/2` where both
[reserved](\`char_reserved?/1\`) and [unreserved characters](\`char_unreserved?/1\`)
are kept unescaped.

#### Examples

    iex> URI.char_unescaped?(?{)
    false

### char_unreserved?(character)

```elixir
@spec char_unreserved?(byte()) :: boolean()
```

Checks if `character` is an unreserved one in a URI.

As specified in [RFC 3986, section 2.3](https://tools.ietf.org/html/rfc3986#section-2.3),
the following characters are unreserved:

- Alphanumeric characters: `A-Z`, `a-z`, `0-9`
- `~`, `_`, `-`, `.`

#### Examples

    iex> URI.char_unreserved?(?_)
    true

### decode(uri)

```elixir
@spec decode(binary()) :: binary()
```

Percent-unescapes a URI.

#### Examples

    iex> URI.decode("https%3A%2F%2Felixir-lang.org")
    "https://elixir-lang.org"

### decode_query(query, map \\ %{}, encoding \\ :www_form)

```elixir
@spec decode_query(binary(), %{optional(binary()) =&gt; binary()}, :rfc3986 | :www_form) ::
  %{
    optional(binary()) =&gt; binary()
  }
```

Decodes `query` into a map.

Given a query string in the form of `key1=value1&key2=value2...`, this
function inserts each key-value pair in the query string as one entry in the
given `map`. Keys and values in the resulting map will be binaries. Keys and
values will be percent-unescaped.

You can specify one of the following `encoding` options:

- `:www_form` - (default, since v1.12.0) keys and values are decoded as per
  `decode_www_form/1`. This is the format typically used by browsers on
  query strings and form data. It decodes "+" as " ".

- `:rfc3986` - (since v1.12.0) keys and values are decoded as per
  `decode/1`. The result is the same as `:www_form` except for leaving "+"
  as is in line with [RFC 3986](https://tools.ietf.org/html/rfc3986).

Encoding defaults to `:www_form` for backward compatibility.

Use `query_decoder/1` if you want to iterate over each value manually.

#### Examples

    iex> URI.decode_query("foo=1&bar=2")
    %{"bar" => "2", "foo" => "1"}
    
    iex> URI.decode_query("percent=oh+yes%21", %{"starting" => "map"})
    %{"percent" => "oh yes!", "starting" => "map"}
    
    iex> URI.decode_query("percent=oh+yes%21", %{}, :rfc3986)
    %{"percent" => "oh+yes!"}

### decode_www_form(string)

```elixir
@spec decode_www_form(binary()) :: binary()
```

Decodes `string` as "x-www-form-urlencoded".

Note "x-www-form-urlencoded" is not specified as part of
RFC 3986. However, it is a commonly used format to encode
query strings and form data by browsers.

#### Examples

    iex> URI.decode_www_form("%3Call+in%2F")
    "<all in/"

### default_port(scheme)

```elixir
@spec default_port(binary()) :: nil | non_neg_integer()
```

Returns the default port for a given `scheme`.

If the scheme is unknown to the `URI` module, this function returns
`nil`. The default port for any scheme can be configured globally
via `default_port/2`.

#### Examples

    iex> URI.default_port("ftp")
    21
    
    iex> URI.default_port("ponzi")
    nil

### default_port(scheme, port)

```elixir
@spec default_port(binary(), non_neg_integer()) :: :ok
```

Registers the default `port` for the given `scheme`.

After this function is called, `port` will be returned by
`default_port/1` for the given scheme `scheme`. Note that this function
changes the default port for the given `scheme` *globally*, meaning for
every application.

It is recommended for this function to be invoked in your
application's start callback in case you want to register
new URIs.

### encode(string, predicate \\ &amp;char_unescaped?/1)

```elixir
@spec encode(binary(), (byte() -&gt; as_boolean(term()))) :: binary()
```

Percent-encodes all characters that require escaping in `string`.

The optional `predicate` argument specifies a function used to detect whether
a byte in the `string` should be escaped:

- if the function returns a truthy value, the byte should be kept as-is.
- if the function returns a falsy value, the byte should be escaped.

The `predicate` argument can use some built-in functions:

- `URI.char_unescaped?/1` (default) - reserved characters (such as `:`
  and `/`) or unreserved (such as letters and numbers) are kept as-is.
  It's typically used to encode the whole URI.
- `URI.char_unreserved?/1` - unreserved characters (such as letters and
  numbers) are kept as-is. It's typically used to encode components in
  a URI, such as query or fragment.
- `URI.char_reserved?/1` - Reserved characters (such as `:` and `/`) are
  kept as-is.

And, you can also use custom functions.

See `encode_www_form/1` if you are interested in encoding `string` as
"x-www-form-urlencoded".

#### Examples

    iex> URI.encode("ftp://s-ite.tld/?value=put it+й")
    "ftp://s-ite.tld/?value=put%20it+%D0%B9"
    
    iex> URI.encode("a string", &(&1 != ?i))
    "a str%69ng"

### encode_query(enumerable, encoding \\ :www_form)

```elixir
@spec encode_query(Enumerable.t(), :rfc3986 | :www_form) :: binary()
```

Encodes `enumerable` into a query string using `encoding`.

Takes an enumerable that enumerates as a list of two-element
tuples (for instance, a map or a keyword list) and returns a string
in the form of `key1=value1&key2=value2...`.

Keys and values can be any term that implements the `String.Chars`
protocol with the exception of lists, which are explicitly forbidden.

You can specify one of the following `encoding` strategies:

- `:www_form` - (default, since v1.12.0) keys and values are URL encoded as
  per `encode_www_form/1`. This is the format typically used by browsers on
  query strings and form data. It encodes " " as "+".

- `:rfc3986` - (since v1.12.0) the same as `:www_form` except it encodes
  " " as "%20" according [RFC 3986](https://tools.ietf.org/html/rfc3986).
  This is the best option if you are encoding in a non-browser situation,
  since encoding spaces as "+" can be ambiguous to URI parsers. This can
  inadvertently lead to spaces being interpreted as literal plus signs.

Encoding defaults to `:www_form` for backward compatibility.

#### Examples

    iex> query = %{"foo" => 1, "bar" => 2}
    iex> URI.encode_query(query)
    "bar=2&foo=1"
    
    iex> query = %{"key" => "value with spaces"}
    iex> URI.encode_query(query)
    "key=value+with+spaces"
    
    iex> query = %{"key" => "value with spaces"}
    iex> URI.encode_query(query, :rfc3986)
    "key=value%20with%20spaces"
    
    iex> URI.encode_query(%{key: [:a, :list]})
    ** (ArgumentError) encode_query/2 values cannot be lists, got: [:a, :list]

### encode_www_form(string)

```elixir
@spec encode_www_form(binary()) :: binary()
```

Encodes `string` as "x-www-form-urlencoded".

Note "x-www-form-urlencoded" is not specified as part of
RFC 3986. However, it is a commonly used format to encode
query strings and form data by browsers.

#### Example

    iex> URI.encode_www_form("put: it+й")
    "put%3A+it%2B%D0%B9"

### merge(uri, rel)

```elixir
@spec merge(t() | binary(), t() | binary()) :: t()
```

Merges two URIs.

This function merges two URIs as per
[RFC 3986, section 5.2](https://tools.ietf.org/html/rfc3986#section-5.2).

#### Examples

    iex> URI.merge(URI.parse("http://google.com"), "/query") |> to_string()
    "http://google.com/query"
    
    iex> URI.merge("http://example.com", "http://google.com") |> to_string()
    "http://google.com"

### new(uri)
*(since 1.13.0)* 
```elixir
@spec new(t() | String.t()) :: {:ok, t()} | {:error, String.t()}
```

Creates a new URI struct from a URI or a string.

If a `%URI{}` struct is given, it returns `{:ok, uri}`. If a string is
given, it will parse and validate it. If the string is valid, it returns
`{:ok, uri}`, otherwise it returns `{:error, part}` with the invalid part
of the URI. For parsing URIs without further validation, see `parse/1`.

This function can parse both absolute and relative URLs. You can check
if a URI is absolute or relative by checking if the `scheme` field is
`nil` or not.

When a URI is given without a port, the value returned by `URI.default_port/1`
for the URI's scheme is used for the `:port` field. The scheme is also
normalized to lowercase.

#### Examples

    iex> URI.new("https://elixir-lang.org/")
    {:ok, %URI{
      fragment: nil,
      host: "elixir-lang.org",
      path: "/",
      port: 443,
      query: nil,
      scheme: "https",
      userinfo: nil
    }}
    
    iex> URI.new("//elixir-lang.org/")
    {:ok, %URI{
      fragment: nil,
      host: "elixir-lang.org",
      path: "/",
      port: nil,
      query: nil,
      scheme: nil,
      userinfo: nil
    }}
    
    iex> URI.new("/foo/bar")
    {:ok, %URI{
      fragment: nil,
      host: nil,
      path: "/foo/bar",
      port: nil,
      query: nil,
      scheme: nil,
      userinfo: nil
    }}
    
    iex> URI.new("foo/bar")
    {:ok, %URI{
      fragment: nil,
      host: nil,
      path: "foo/bar",
      port: nil,
      query: nil,
      scheme: nil,
      userinfo: nil
    }}
    
    iex> URI.new("//[fe80::]/")
    {:ok, %URI{
      fragment: nil,
      host: "fe80::",
      path: "/",
      port: nil,
      query: nil,
      scheme: nil,
      userinfo: nil
    }}
    
    iex> URI.new("https:?query")
    {:ok, %URI{
      fragment: nil,
      host: nil,
      path: nil,
      port: 443,
      query: "query",
      scheme: "https",
      userinfo: nil
    }}
    
    iex> URI.new("/invalid_greater_than_in_path/>")
    {:error, ">"}

Giving an existing URI simply returns it wrapped in a tuple:

    iex> {:ok, uri} = URI.new("https://elixir-lang.org/")
    iex> URI.new(uri)
    {:ok, %URI{
      fragment: nil,
      host: "elixir-lang.org",
      path: "/",
      port: 443,
      query: nil,
      scheme: "https",
      userinfo: nil
    }}

### new!(uri)
*(since 1.13.0)* 
```elixir
@spec new!(t() | String.t()) :: t()
```

Similar to `new/1` but raises `URI.Error` if an invalid string is given.

#### Examples

    iex> URI.new!("https://elixir-lang.org/")
    %URI{
      fragment: nil,
      host: "elixir-lang.org",
      path: "/",
      port: 443,
      query: nil,
      scheme: "https",
      userinfo: nil
    }
    
    iex> URI.new!("/invalid_greater_than_in_path/>")
    ** (URI.Error) cannot parse due to reason invalid_uri: ">"

Giving an existing URI simply returns it:

    iex> uri = URI.new!("https://elixir-lang.org/")
    iex> URI.new!(uri)
    %URI{
      fragment: nil,
      host: "elixir-lang.org",
      path: "/",
      port: 443,
      query: nil,
      scheme: "https",
      userinfo: nil
    }

### parse(uri)

```elixir
@spec parse(t() | binary()) :: t()
```

Parses a URI into its components, without further validation.

This function can parse both absolute and relative URLs. You can check
if a URI is absolute or relative by checking if the `scheme` field is
nil or not. Furthermore, this function expects both absolute and
relative URIs to be well-formed and does not perform any validation.
See the "Examples" section below. Use `new/1` if you want to validate
the URI fields after parsing.

When a URI is given without a port, the value returned by `URI.default_port/1`
for the URI's scheme is used for the `:port` field. The scheme is also
normalized to lowercase.

If a `%URI{}` struct is given to this function, this function returns it
unmodified.

> #### `:authority` field {: .info}
> 
> This function sets the deprecated field `:authority` for backwards-compatibility reasons.

#### Examples

    iex> URI.parse("https://elixir-lang.org/")
    %URI{
      authority: "elixir-lang.org",
      fragment: nil,
      host: "elixir-lang.org",
      path: "/",
      port: 443,
      query: nil,
      scheme: "https",
      userinfo: nil
    }
    
    iex> URI.parse("//elixir-lang.org/")
    %URI{
      authority: "elixir-lang.org",
      fragment: nil,
      host: "elixir-lang.org",
      path: "/",
      port: nil,
      query: nil,
      scheme: nil,
      userinfo: nil
    }
    
    iex> URI.parse("/foo/bar")
    %URI{
      fragment: nil,
      host: nil,
      path: "/foo/bar",
      port: nil,
      query: nil,
      scheme: nil,
      userinfo: nil
    }
    
    iex> URI.parse("foo/bar")
    %URI{
      fragment: nil,
      host: nil,
      path: "foo/bar",
      port: nil,
      query: nil,
      scheme: nil,
      userinfo: nil
    }

In contrast to `URI.new/1`, this function will parse poorly-formed
URIs, for example:

    iex> URI.parse("/invalid_greater_than_in_path/>")
    %URI{
      fragment: nil,
      host: nil,
      path: "/invalid_greater_than_in_path/>",
      port: nil,
      query: nil,
      scheme: nil,
      userinfo: nil
    }

Another example is a URI with brackets in query strings. It is accepted
by `parse/1`, it is commonly accepted by browsers, but it will be refused
by `new/1`:

    iex> URI.parse("/?foo[bar]=baz")
    %URI{
      fragment: nil,
      host: nil,
      path: "/",
      port: nil,
      query: "foo[bar]=baz",
      scheme: nil,
      userinfo: nil
    }

### query_decoder(query, encoding \\ :www_form)

```elixir
@spec query_decoder(binary(), :rfc3986 | :www_form) :: Enumerable.t()
```

Returns a stream of two-element tuples representing key-value pairs in the
given `query`.

Key and value in each tuple will be binaries and will be percent-unescaped.

You can specify one of the following `encoding` options:

- `:www_form` - (default, since v1.12.0) keys and values are decoded as per
  `decode_www_form/1`. This is the format typically used by browsers on
  query strings and form data. It decodes "+" as " ".

- `:rfc3986` - (since v1.12.0) keys and values are decoded as per
  `decode/1`. The result is the same as `:www_form` except for leaving "+"
  as is in line with [RFC 3986](https://tools.ietf.org/html/rfc3986).

Encoding defaults to `:www_form` for backward compatibility.

#### Examples

    iex> URI.query_decoder("foo=1&bar=2") |> Enum.to_list()
    [{"foo", "1"}, {"bar", "2"}]
    
    iex> URI.query_decoder("food=bread%26butter&drinks=tap%20water+please") |> Enum.to_list()
    [{"food", "bread&butter"}, {"drinks", "tap water please"}]
    
    iex> URI.query_decoder("food=bread%26butter&drinks=tap%20water+please", :rfc3986) |> Enum.to_list()
    [{"food", "bread&butter"}, {"drinks", "tap water+please"}]

### to_string(uri)

```elixir
@spec to_string(t()) :: binary()
```

Returns the string representation of the given [URI struct](\`t:t/0\`).

#### Examples

    iex> uri = URI.parse("http://google.com")
    iex> URI.to_string(uri)
    "http://google.com"
    
    iex> uri = URI.parse("foo://bar.baz")
    iex> URI.to_string(uri)
    "foo://bar.baz"



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
