# Base 
(Elixir v1.18.0-dev)

This module provides data encoding and decoding functions
according to [RFC 4648](https://tools.ietf.org/html/rfc4648).

This document defines the commonly used base 16, base 32, and base
64 encoding schemes.

## Base 16 alphabet

| Value | Encoding | Value | Encoding | Value | Encoding | Value | Encoding |
|------:|:---------|------:|:---------|------:|:---------|------:|:---------|
|     0 | 0        |     4 | 4        |     8 | 8        |    12 | C        |
|     1 | 1        |     5 | 5        |     9 | 9        |    13 | D        |
|     2 | 2        |     6 | 6        |    10 | A        |    14 | E        |
|     3 | 3        |     7 | 7        |    11 | B        |    15 | F        |

## Base 32 alphabet

| Value | Encoding | Value | Encoding | Value | Encoding | Value | Encoding |
|------:|:---------|------:|:---------|------:|:---------|------:|:---------|
|     0 | A        |     9 | J        |    18 | S        |    27 | 3        |
|     1 | B        |    10 | K        |    19 | T        |    28 | 4        |
|     2 | C        |    11 | L        |    20 | U        |    29 | 5        |
|     3 | D        |    12 | M        |    21 | V        |    30 | 6        |
|     4 | E        |    13 | N        |    22 | W        |    31 | 7        |
|     5 | F        |    14 | O        |    23 | X        |       |          |
|     6 | G        |    15 | P        |    24 | Y        | (pad) | =        |
|     7 | H        |    16 | Q        |    25 | Z        |       |          |
|     8 | I        |    17 | R        |    26 | 2        |       |          |

## Base 32 (extended hex) alphabet

| Value | Encoding | Value | Encoding | Value | Encoding | Value | Encoding |
|------:|:---------|------:|:---------|------:|:---------|------:|:---------|
|     0 | 0        |     9 | 9        |    18 | I        |    27 | R        |
|     1 | 1        |    10 | A        |    19 | J        |    28 | S        |
|     2 | 2        |    11 | B        |    20 | K        |    29 | T        |
|     3 | 3        |    12 | C        |    21 | L        |    30 | U        |
|     4 | 4        |    13 | D        |    22 | M        |    31 | V        |
|     5 | 5        |    14 | E        |    23 | N        |       |          |
|     6 | 6        |    15 | F        |    24 | O        | (pad) | =        |
|     7 | 7        |    16 | G        |    25 | P        |       |          |
|     8 | 8        |    17 | H        |    26 | Q        |       |          |

## Base 64 alphabet

| Value |  Encoding | Value | Encoding | Value | Encoding | Value | Encoding |
|------:|:----------|------:|:---------|------:|:---------|------:|:---------|
|     0 | A         |    17 | R        |    34 | i        |    51 | z        |
|     1 | B         |    18 | S        |    35 | j        |    52 | 0        |
|     2 | C         |    19 | T        |    36 | k        |    53 | 1        |
|     3 | D         |    20 | U        |    37 | l        |    54 | 2        |
|     4 | E         |    21 | V        |    38 | m        |    55 | 3        |
|     5 | F         |    22 | W        |    39 | n        |    56 | 4        |
|     6 | G         |    23 | X        |    40 | o        |    57 | 5        |
|     7 | H         |    24 | Y        |    41 | p        |    58 | 6        |
|     8 | I         |    25 | Z        |    42 | q        |    59 | 7        |
|     9 | J         |    26 | a        |    43 | r        |    60 | 8        |
|    10 | K         |    27 | b        |    44 | s        |    61 | 9        |
|    11 | L         |    28 | c        |    45 | t        |    62 | +        |
|    12 | M         |    29 | d        |    46 | u        |    63 | /        |
|    13 | N         |    30 | e        |    47 | v        |       |          |
|    14 | O         |    31 | f        |    48 | w        | (pad) | =        |
|    15 | P         |    32 | g        |    49 | x        |       |          |
|    16 | Q         |    33 | h        |    50 | y        |       |          |

## Base 64 (URL and filename safe) alphabet

| Value | Encoding | Value | Encoding | Value | Encoding | Value | Encoding |
|------:|:---------|------:|:---------|------:|:---------|------:|:---------|
|     0 | A        |    17 | R        |    34 | i        |    51 | z        |
|     1 | B        |    18 | S        |    35 | j        |    52 | 0        |
|     2 | C        |    19 | T        |    36 | k        |    53 | 1        |
|     3 | D        |    20 | U        |    37 | l        |    54 | 2        |
|     4 | E        |    21 | V        |    38 | m        |    55 | 3        |
|     5 | F        |    22 | W        |    39 | n        |    56 | 4        |
|     6 | G        |    23 | X        |    40 | o        |    57 | 5        |
|     7 | H        |    24 | Y        |    41 | p        |    58 | 6        |
|     8 | I        |    25 | Z        |    42 | q        |    59 | 7        |
|     9 | J        |    26 | a        |    43 | r        |    60 | 8        |
|    10 | K        |    27 | b        |    44 | s        |    61 | 9        |
|    11 | L        |    28 | c        |    45 | t        |    62 | -        |
|    12 | M        |    29 | d        |    46 | u        |    63 | \_        |
|    13 | N        |    30 | e        |    47 | v        |       |          |
|    14 | O        |    31 | f        |    48 | w        | (pad) | =        |
|    15 | P        |    32 | g        |    49 | x        |       |          |
|    16 | Q        |    33 | h        |    50 | y        |       |          |

## Types

### decode_case()

```elixir
@type decode_case() :: :upper | :lower | :mixed
```



### encode_case()

```elixir
@type encode_case() :: :upper | :lower
```



## Functions

### decode16(string, opts \\ [])

```elixir
@spec decode16(binary(), [{:case, decode_case()}]) :: {:ok, binary()} | :error
```

Decodes a base 16 encoded string into a binary string.

#### Options

The accepted options are:

- `:case` - specifies the character case to accept when decoding

The values for `:case` can be:

- `:upper` - only allows upper case characters (default)
- `:lower` - only allows lower case characters
- `:mixed` - allows mixed case characters

#### Examples

    iex> Base.decode16("666F6F626172")
    {:ok, "foobar"}
    
    iex> Base.decode16("666f6f626172", case: :lower)
    {:ok, "foobar"}
    
    iex> Base.decode16("666f6F626172", case: :mixed)
    {:ok, "foobar"}

### decode16!(string, opts \\ [])

```elixir
@spec decode16!(binary(), [{:case, decode_case()}]) :: binary()
```

Decodes a base 16 encoded string into a binary string.

#### Options

The accepted options are:

- `:case` - specifies the character case to accept when decoding

The values for `:case` can be:

- `:upper` - only allows upper case characters (default)
- `:lower` - only allows lower case characters
- `:mixed` - allows mixed case characters

An `ArgumentError` exception is raised if the padding is incorrect or
a non-alphabet character is present in the string.

#### Examples

    iex> Base.decode16!("666F6F626172")
    "foobar"
    
    iex> Base.decode16!("666f6f626172", case: :lower)
    "foobar"
    
    iex> Base.decode16!("666f6F626172", case: :mixed)
    "foobar"

### decode32(string, opts \\ [])

```elixir
@spec decode32(binary(), case: decode_case(), padding: boolean()) ::
  {:ok, binary()} | :error
```

Decodes a base 32 encoded string into a binary string.

#### Options

The accepted options are:

- `:case` - specifies the character case to accept when decoding
- `:padding` - specifies whether to require padding

The values for `:case` can be:

- `:upper` - only allows  upper case characters (default)
- `:lower` - only allows lower case characters
- `:mixed` - allows mixed case characters

The values for `:padding` can be:

- `true` - requires the input string to be padded to the nearest multiple of 8 (default)
- `false` - ignores padding from the input string

#### Examples

    iex> Base.decode32("MZXW6YTBOI======")
    {:ok, "foobar"}
    
    iex> Base.decode32("mzxw6ytboi======", case: :lower)
    {:ok, "foobar"}
    
    iex> Base.decode32("mzXW6ytBOi======", case: :mixed)
    {:ok, "foobar"}
    
    iex> Base.decode32("MZXW6YTBOI", padding: false)
    {:ok, "foobar"}

### decode32!(string, opts \\ [])

```elixir
@spec decode32!(binary(), case: decode_case(), padding: boolean()) :: binary()
```

Decodes a base 32 encoded string into a binary string.

An `ArgumentError` exception is raised if the padding is incorrect or
a non-alphabet character is present in the string.

#### Options

The accepted options are:

- `:case` - specifies the character case to accept when decoding
- `:padding` - specifies whether to require padding

The values for `:case` can be:

- `:upper` - only allows upper case characters (default)
- `:lower` - only allows lower case characters
- `:mixed` - allows mixed case characters

The values for `:padding` can be:

- `true` - requires the input string to be padded to the nearest multiple of 8 (default)
- `false` - ignores padding from the input string

#### Examples

    iex> Base.decode32!("MZXW6YTBOI======")
    "foobar"
    
    iex> Base.decode32!("mzxw6ytboi======", case: :lower)
    "foobar"
    
    iex> Base.decode32!("mzXW6ytBOi======", case: :mixed)
    "foobar"
    
    iex> Base.decode32!("MZXW6YTBOI", padding: false)
    "foobar"

### decode64(string, opts \\ [])

```elixir
@spec decode64(binary(), ignore: :whitespace, padding: boolean()) ::
  {:ok, binary()} | :error
```

Decodes a base 64 encoded string into a binary string.

Accepts `ignore: :whitespace` option which will ignore all the
whitespace characters in the input string.

Accepts `padding: false` option which will ignore padding from
the input string.

#### Examples

    iex> Base.decode64("Zm9vYmFy")
    {:ok, "foobar"}
    
    iex> Base.decode64("Zm9vYmFy\n", ignore: :whitespace)
    {:ok, "foobar"}
    
    iex> Base.decode64("Zm9vYg==")
    {:ok, "foob"}
    
    iex> Base.decode64("Zm9vYg", padding: false)
    {:ok, "foob"}

### decode64!(string, opts \\ [])

```elixir
@spec decode64!(binary(), ignore: :whitespace, padding: boolean()) :: binary()
```

Decodes a base 64 encoded string into a binary string.

Accepts `ignore: :whitespace` option which will ignore all the
whitespace characters in the input string.

Accepts `padding: false` option which will ignore padding from
the input string.

An `ArgumentError` exception is raised if the padding is incorrect or
a non-alphabet character is present in the string.

#### Examples

    iex> Base.decode64!("Zm9vYmFy")
    "foobar"
    
    iex> Base.decode64!("Zm9vYmFy\n", ignore: :whitespace)
    "foobar"
    
    iex> Base.decode64!("Zm9vYg==")
    "foob"
    
    iex> Base.decode64!("Zm9vYg", padding: false)
    "foob"

### encode16(data, opts \\ [])

```elixir
@spec encode16(binary(), [{:case, encode_case()}]) :: binary()
```

Encodes a binary string into a base 16 encoded string.

#### Options

The accepted options are:

- `:case` - specifies the character case to use when encoding

The values for `:case` can be:

- `:upper` - uses upper case characters (default)
- `:lower` - uses lower case characters

#### Examples

    iex> Base.encode16("foobar")
    "666F6F626172"
    
    iex> Base.encode16("foobar", case: :lower)
    "666f6f626172"

### encode32(data, opts \\ [])

```elixir
@spec encode32(binary(), case: encode_case(), padding: boolean()) :: binary()
```

Encodes a binary string into a base 32 encoded string.

#### Options

The accepted options are:

- `:case` - specifies the character case to use when encoding
- `:padding` - specifies whether to apply padding

The values for `:case` can be:

- `:upper` - uses upper case characters (default)
- `:lower` - uses lower case characters

The values for `:padding` can be:

- `true` - pad the output string to the nearest multiple of 8 (default)
- `false` - omit padding from the output string

#### Examples

    iex> Base.encode32("foobar")
    "MZXW6YTBOI======"
    
    iex> Base.encode32("foobar", case: :lower)
    "mzxw6ytboi======"
    
    iex> Base.encode32("foobar", padding: false)
    "MZXW6YTBOI"

### encode64(data, opts \\ [])

```elixir
@spec encode64(binary(), [{:padding, boolean()}]) :: binary()
```

Encodes a binary string into a base 64 encoded string.

Accepts `padding: false` option which will omit padding from
the output string.

#### Examples

    iex> Base.encode64("foobar")
    "Zm9vYmFy"
    
    iex> Base.encode64("foob")
    "Zm9vYg=="
    
    iex> Base.encode64("foob", padding: false)
    "Zm9vYg"

### hex_decode32(string, opts \\ [])

```elixir
@spec hex_decode32(binary(), case: decode_case(), padding: boolean()) ::
  {:ok, binary()} | :error
```

Decodes a base 32 encoded string with extended hexadecimal alphabet
into a binary string.

#### Options

The accepted options are:

- `:case` - specifies the character case to accept when decoding
- `:padding` - specifies whether to require padding

The values for `:case` can be:

- `:upper` - only allows upper case characters (default)
- `:lower` - only allows lower case characters
- `:mixed` - allows mixed case characters

The values for `:padding` can be:

- `true` - requires the input string to be padded to the nearest multiple of 8 (default)
- `false` - ignores padding from the input string

#### Examples

    iex> Base.hex_decode32("CPNMUOJ1E8======")
    {:ok, "foobar"}
    
    iex> Base.hex_decode32("cpnmuoj1e8======", case: :lower)
    {:ok, "foobar"}
    
    iex> Base.hex_decode32("cpnMuOJ1E8======", case: :mixed)
    {:ok, "foobar"}
    
    iex> Base.hex_decode32("CPNMUOJ1E8", padding: false)
    {:ok, "foobar"}

### hex_decode32!(string, opts \\ [])

```elixir
@spec hex_decode32!(binary(), case: decode_case(), padding: boolean()) :: binary()
```

Decodes a base 32 encoded string with extended hexadecimal alphabet
into a binary string.

An `ArgumentError` exception is raised if the padding is incorrect or
a non-alphabet character is present in the string.

#### Options

The accepted options are:

- `:case` - specifies the character case to accept when decoding
- `:padding` - specifies whether to require padding

The values for `:case` can be:

- `:upper` - only allows upper case characters (default)
- `:lower` - only allows lower case characters
- `:mixed` - allows mixed case characters

The values for `:padding` can be:

- `true` - requires the input string to be padded to the nearest multiple of 8 (default)
- `false` - ignores padding from the input string

#### Examples

    iex> Base.hex_decode32!("CPNMUOJ1E8======")
    "foobar"
    
    iex> Base.hex_decode32!("cpnmuoj1e8======", case: :lower)
    "foobar"
    
    iex> Base.hex_decode32!("cpnMuOJ1E8======", case: :mixed)
    "foobar"
    
    iex> Base.hex_decode32!("CPNMUOJ1E8", padding: false)
    "foobar"

### hex_encode32(data, opts \\ [])

```elixir
@spec hex_encode32(binary(), case: encode_case(), padding: boolean()) :: binary()
```

Encodes a binary string into a base 32 encoded string with an
extended hexadecimal alphabet.

#### Options

The accepted options are:

- `:case` - specifies the character case to use when encoding
- `:padding` - specifies whether to apply padding

The values for `:case` can be:

- `:upper` - uses upper case characters (default)
- `:lower` - uses lower case characters

The values for `:padding` can be:

- `true` - pad the output string to the nearest multiple of 8 (default)
- `false` - omit padding from the output string

#### Examples

    iex> Base.hex_encode32("foobar")
    "CPNMUOJ1E8======"
    
    iex> Base.hex_encode32("foobar", case: :lower)
    "cpnmuoj1e8======"
    
    iex> Base.hex_encode32("foobar", padding: false)
    "CPNMUOJ1E8"

### url_decode64(string, opts \\ [])

```elixir
@spec url_decode64(binary(), ignore: :whitespace, padding: boolean()) ::
  {:ok, binary()} | :error
```

Decodes a base 64 encoded string with URL and filename safe alphabet
into a binary string.

Accepts `ignore: :whitespace` option which will ignore all the
whitespace characters in the input string.

Accepts `padding: false` option which will ignore padding from
the input string.

#### Examples

    iex> Base.url_decode64("_3_-_A==")
    {:ok, <<255, 127, 254, 252>>}
    
    iex> Base.url_decode64("_3_-_A==\n", ignore: :whitespace)
    {:ok, <<255, 127, 254, 252>>}
    
    iex> Base.url_decode64("_3_-_A", padding: false)
    {:ok, <<255, 127, 254, 252>>}

### url_decode64!(string, opts \\ [])

```elixir
@spec url_decode64!(binary(), ignore: :whitespace, padding: boolean()) :: binary()
```

Decodes a base 64 encoded string with URL and filename safe alphabet
into a binary string.

Accepts `ignore: :whitespace` option which will ignore all the
whitespace characters in the input string.

Accepts `padding: false` option which will ignore padding from
the input string.

An `ArgumentError` exception is raised if the padding is incorrect or
a non-alphabet character is present in the string.

#### Examples

    iex> Base.url_decode64!("_3_-_A==")
    <<255, 127, 254, 252>>
    
    iex> Base.url_decode64!("_3_-_A==\n", ignore: :whitespace)
    <<255, 127, 254, 252>>
    
    iex> Base.url_decode64!("_3_-_A", padding: false)
    <<255, 127, 254, 252>>

### url_encode64(data, opts \\ [])

```elixir
@spec url_encode64(binary(), [{:padding, boolean()}]) :: binary()
```

Encodes a binary string into a base 64 encoded string with URL and filename
safe alphabet.

Accepts `padding: false` option which will omit padding from
the output string.

#### Examples

    iex> Base.url_encode64(<<255, 127, 254, 252>>)
    "_3_-_A=="
    
    iex> Base.url_encode64(<<255, 127, 254, 252>>, padding: false)
    "_3_-_A"



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
