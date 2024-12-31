# Integer 
(Elixir v1.18.0-dev)

Functions for working with integers.

Some functions that work on integers are found in `Kernel`:

- `Kernel.abs/1`
- `Kernel.div/2`
- `Kernel.max/2`
- `Kernel.min/2`
- `Kernel.rem/2`


## Guards

### is_even(integer)
*(macro)* 


Determines if an `integer` is even.

Returns `true` if the given `integer` is an even number,
otherwise it returns `false`.

Allowed in guard clauses.

#### Examples

    iex> Integer.is_even(10)
    true
    
    iex> Integer.is_even(5)
    false
    
    iex> Integer.is_even(-10)
    true
    
    iex> Integer.is_even(0)
    true


### is_odd(integer)
*(macro)* 


Determines if `integer` is odd.

Returns `true` if the given `integer` is an odd number,
otherwise it returns `false`.

Allowed in guard clauses.

#### Examples

    iex> Integer.is_odd(5)
    true
    
    iex> Integer.is_odd(6)
    false
    
    iex> Integer.is_odd(-5)
    true
    
    iex> Integer.is_odd(0)
    false


## Functions

### digits(integer, base \\ 10)

```elixir
@spec digits(integer(), pos_integer()) :: [integer(), ...]
```

Returns the ordered digits for the given `integer`.

An optional `base` value may be provided representing the radix for the returned
digits. This one must be an integer \>= 2.

#### Examples

    iex> Integer.digits(123)
    [1, 2, 3]
    
    iex> Integer.digits(170, 2)
    [1, 0, 1, 0, 1, 0, 1, 0]
    
    iex> Integer.digits(-170, 2)
    [-1, 0, -1, 0, -1, 0, -1, 0]


### extended_gcd(a, b)
*(since 1.12.0)* 
```elixir
@spec extended_gcd(integer(), integer()) :: {non_neg_integer(), integer(), integer()}
```

Returns the extended greatest common divisor of the two given integers.

This function uses the extended Euclidean algorithm to return a three-element tuple with the `gcd`
and the coefficients `m` and `n` of Bézout's identity such that:

    gcd(a, b) = m*a + n*b

By convention, `extended_gcd(0, 0)` returns `{0, 0, 0}`.

#### Examples

    iex> Integer.extended_gcd(240, 46)
    {2, -9, 47}
    iex> Integer.extended_gcd(46, 240)
    {2, 47, -9}
    iex> Integer.extended_gcd(-46, 240)
    {2, -47, -9}
    iex> Integer.extended_gcd(-46, -240)
    {2, -47, 9}
    
    iex> Integer.extended_gcd(14, 21)
    {7, -1, 1}
    
    iex> Integer.extended_gcd(10, 0)
    {10, 1, 0}
    iex> Integer.extended_gcd(0, 10)
    {10, 0, 1}
    iex> Integer.extended_gcd(0, 0)
    {0, 0, 0}


### floor_div(dividend, divisor)
*(since 1.4.0)* 
```elixir
@spec floor_div(integer(), neg_integer() | pos_integer()) :: integer()
```

Performs a floored integer division.

Raises an `ArithmeticError` exception if one of the arguments is not an
integer, or when the `divisor` is `0`.

This function performs a *floored* integer division, which means that
the result will always be rounded towards negative infinity.

If you want to perform truncated integer division (rounding towards zero),
use `Kernel.div/2` instead.

#### Examples

    iex> Integer.floor_div(5, 2)
    2
    iex> Integer.floor_div(6, -4)
    -2
    iex> Integer.floor_div(-99, 2)
    -50


### gcd(integer1, integer2)
*(since 1.5.0)* 
```elixir
@spec gcd(integer(), integer()) :: non_neg_integer()
```

Returns the greatest common divisor of the two given integers.

The greatest common divisor (GCD) of `integer1` and `integer2` is the largest positive
integer that divides both `integer1` and `integer2` without leaving a remainder.

By convention, `gcd(0, 0)` returns `0`.

#### Examples

    iex> Integer.gcd(2, 3)
    1
    
    iex> Integer.gcd(8, 12)
    4
    
    iex> Integer.gcd(8, -12)
    4
    
    iex> Integer.gcd(10, 0)
    10
    
    iex> Integer.gcd(7, 7)
    7
    
    iex> Integer.gcd(0, 0)
    0


### mod(dividend, divisor)
*(since 1.4.0)* 
```elixir
@spec mod(integer(), neg_integer() | pos_integer()) :: integer()
```

Computes the modulo remainder of an integer division.

This function performs a [floored division](\`floor_div/2\`), which means that
the result will always have the sign of the `divisor`.

Raises an `ArithmeticError` exception if one of the arguments is not an
integer, or when the `divisor` is `0`.

#### Examples

    iex> Integer.mod(5, 2)
    1
    iex> Integer.mod(6, -4)
    -2


### parse(binary, base \\ 10)

```elixir
@spec parse(binary(), 2..36) :: {integer(), remainder_of_binary :: binary()} | :error
```

Parses a text representation of an integer.

An optional `base` to the corresponding integer can be provided.
If `base` is not given, 10 will be used.

If successful, returns a tuple in the form of `{integer, remainder_of_binary}`.
Otherwise `:error`.

Raises an error if `base` is less than 2 or more than 36.

If you want to convert a string-formatted integer directly to an integer,
`String.to_integer/1` or `String.to_integer/2` can be used instead.

#### Examples

    iex> Integer.parse("34")
    {34, ""}
    
    iex> Integer.parse("34.5")
    {34, ".5"}
    
    iex> Integer.parse("three")
    :error
    
    iex> Integer.parse("34", 10)
    {34, ""}
    
    iex> Integer.parse("f4", 16)
    {244, ""}
    
    iex> Integer.parse("Awww++", 36)
    {509216, "++"}
    
    iex> Integer.parse("fab", 10)
    :error
    
    iex> Integer.parse("a2", 38)
    ** (ArgumentError) invalid base 38


### pow(base, exponent)
*(since 1.12.0)* 
```elixir
@spec pow(integer(), non_neg_integer()) :: integer()
```

Computes `base` raised to power of `exponent`.

Both `base` and `exponent` must be integers.
The exponent must be zero or positive.

See `Float.pow/2` for exponentiation of negative
exponents as well as floats.

#### Examples

    iex> Integer.pow(2, 0)
    1
    iex> Integer.pow(2, 1)
    2
    iex> Integer.pow(2, 10)
    1024
    iex> Integer.pow(2, 11)
    2048
    iex> Integer.pow(2, 64)
    0x10000000000000000
    
    iex> Integer.pow(3, 4)
    81
    iex> Integer.pow(4, 3)
    64
    
    iex> Integer.pow(-2, 3)
    -8
    iex> Integer.pow(-2, 4)
    16
    
    iex> Integer.pow(2, -2)
    ** (ArithmeticError) bad argument in arithmetic expression


### to_charlist(integer, base \\ 10)

```elixir
@spec to_charlist(integer(), 2..36) :: charlist()
```

Returns a charlist which corresponds to the text representation
of `integer` in the given `base`.

`base` can be an integer between 2 and 36. If no `base` is given,
it defaults to `10`.

Inlined by the compiler.

#### Examples

    iex> Integer.to_charlist(123)
    ~c"123"
    
    iex> Integer.to_charlist(+456)
    ~c"456"
    
    iex> Integer.to_charlist(-789)
    ~c"-789"
    
    iex> Integer.to_charlist(0123)
    ~c"123"
    
    iex> Integer.to_charlist(100, 16)
    ~c"64"
    
    iex> Integer.to_charlist(-100, 16)
    ~c"-64"
    
    iex> Integer.to_charlist(882_681_651, 36)
    ~c"ELIXIR"


### to_string(integer, base \\ 10)

```elixir
@spec to_string(integer(), 2..36) :: String.t()
```

Returns a binary which corresponds to the text representation
of `integer` in the given `base`.

`base` can be an integer between 2 and 36. If no `base` is given,
it defaults to `10`.

Inlined by the compiler.

#### Examples

    iex> Integer.to_string(123)
    "123"
    
    iex> Integer.to_string(+456)
    "456"
    
    iex> Integer.to_string(-789)
    "-789"
    
    iex> Integer.to_string(0123)
    "123"
    
    iex> Integer.to_string(100, 16)
    "64"
    
    iex> Integer.to_string(-100, 16)
    "-64"
    
    iex> Integer.to_string(882_681_651, 36)
    "ELIXIR"


### undigits(digits, base \\ 10)

```elixir
@spec undigits([integer()], pos_integer()) :: integer()
```

Returns the integer represented by the ordered `digits`.

An optional `base` value may be provided representing the radix for the `digits`.
Base has to be an integer greater than or equal to `2`.

#### Examples

    iex> Integer.undigits([1, 2, 3])
    123
    
    iex> Integer.undigits([1, 4], 16)
    20
    
    iex> Integer.undigits([])
    0




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
