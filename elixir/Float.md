# Float 
(Elixir v1.18.0-dev)

Functions for working with floating-point numbers.

For mathematical operations on top of floating-points,
see Erlang's [`:math`](\`:math\`) module.

## Kernel functions

There are functions related to floating-point numbers on the `Kernel` module
too. Here is a list of them:

- `Kernel.round/1`: rounds a number to the nearest integer.
- `Kernel.trunc/1`: returns the integer part of a number.

## Known issues

There are some very well known problems with floating-point numbers
and arithmetic due to the fact most decimal fractions cannot be
represented by a floating-point binary and most operations are not exact,
but operate on approximations. Those issues are not specific
to Elixir, they are a property of floating point representation itself.

For example, the numbers 0.1 and 0.01 are two of them, what means the result
of squaring 0.1 does not give 0.01 neither the closest representable. Here is
what happens in this case:

- The closest representable number to 0.1 is 0.1000000014
- The closest representable number to 0.01 is 0.0099999997
- Doing 0.1 \* 0.1 should return 0.01, but because 0.1 is actually 0.1000000014,
  the result is 0.010000000000000002, and because this is not the closest
  representable number to 0.01, you'll get the wrong result for this operation

There are also other known problems like flooring or rounding numbers. See
`round/2` and `floor/2` for more details about them.

To learn more about floating-point arithmetic visit:

- [0.30000000000000004.com](http://0.30000000000000004.com/)
- [What Every Programmer Should Know About Floating-Point Arithmetic](https://floating-point-gui.de/)

## Types

### precision_range()

```elixir
@type precision_range() :: 0..15
```



## Functions

### ceil(number, precision \\ 0)

```elixir
@spec ceil(float(), precision_range()) :: float()
```

Rounds a float to the smallest float greater than or equal to `number`.

`ceil/2` also accepts a precision to round a floating-point value down
to an arbitrary number of fractional digits (between 0 and 15).

The operation is performed on the binary floating point, without a
conversion to decimal.

The behavior of `ceil/2` for floats can be surprising. For example:

    iex> Float.ceil(-12.52, 2)
    -12.51

One may have expected it to ceil to -12.52. This is not a bug.
Most decimal fractions cannot be represented as a binary floating point
and therefore the number above is internally represented as -12.51999999,
which explains the behavior above.

This function always returns floats. `Kernel.trunc/1` may be used instead to
truncate the result to an integer afterwards.

#### Examples

    iex> Float.ceil(34.25)
    35.0
    iex> Float.ceil(-56.5)
    -56.0
    iex> Float.ceil(34.251, 2)
    34.26
    iex> Float.ceil(-0.01)
    -0.0

### floor(number, precision \\ 0)

```elixir
@spec floor(float(), precision_range()) :: float()
```

Rounds a float to the largest float less than or equal to `number`.

`floor/2` also accepts a precision to round a floating-point value down
to an arbitrary number of fractional digits (between 0 and 15).
The operation is performed on the binary floating point, without a
conversion to decimal.

This function always returns a float. `Kernel.trunc/1` may be used instead to
truncate the result to an integer afterwards.

#### Known issues

The behavior of `floor/2` for floats can be surprising. For example:

    iex> Float.floor(12.52, 2)
    12.51

One may have expected it to floor to 12.52. This is not a bug.
Most decimal fractions cannot be represented as a binary floating point
and therefore the number above is internally represented as 12.51999999,
which explains the behavior above.

#### Examples

    iex> Float.floor(34.25)
    34.0
    iex> Float.floor(-56.5)
    -57.0
    iex> Float.floor(34.259, 2)
    34.25

### max_finite()

```elixir
@spec max_finite() :: float()
```

Returns the maximum finite value for a float.

#### Examples

    iex> Float.max_finite()
    1.7976931348623157e308

### min_finite()

```elixir
@spec min_finite() :: float()
```

Returns the minimum finite value for a float.

#### Examples

    iex> Float.min_finite()
    -1.7976931348623157e308

### parse(binary)

```elixir
@spec parse(binary()) :: {float(), binary()} | :error
```

Parses a binary into a float.

If successful, returns a tuple in the form of `{float, remainder_of_binary}`;
when the binary cannot be coerced into a valid float, the atom `:error` is
returned.

If the size of float exceeds the maximum size of `1.7976931348623157e+308`,
`:error` is returned even though the textual representation itself might be
well formed.

If you want to convert a string-formatted float directly to a float,
`String.to_float/1` can be used instead.

#### Examples

    iex> Float.parse("34")
    {34.0, ""}
    iex> Float.parse("34.25")
    {34.25, ""}
    iex> Float.parse("56.5xyz")
    {56.5, "xyz"}
    
    iex> Float.parse(".12")
    :error
    iex> Float.parse("pi")
    :error
    iex> Float.parse("1.7976931348623159e+308")
    :error

### pow(base, exponent)
*(since 1.12.0)* 
```elixir
@spec pow(float(), number()) :: float()
```

Computes `base` raised to power of `exponent`.

`base` must be a float and `exponent` can be any number.
However, if a negative base and a fractional exponent
are given, it raises `ArithmeticError`.

It always returns a float. See `Integer.pow/2` for
exponentiation that returns integers.

#### Examples

    iex> Float.pow(2.0, 0)
    1.0
    iex> Float.pow(2.0, 1)
    2.0
    iex> Float.pow(2.0, 10)
    1024.0
    iex> Float.pow(2.0, -1)
    0.5
    iex> Float.pow(2.0, -3)
    0.125
    
    iex> Float.pow(3.0, 1.5)
    5.196152422706632
    
    iex> Float.pow(-2.0, 3)
    -8.0
    iex> Float.pow(-2.0, 4)
    16.0
    
    iex> Float.pow(-1.0, 0.5)
    ** (ArithmeticError) bad argument in arithmetic expression

### ratio(float)
*(since 1.4.0)* 
```elixir
@spec ratio(float()) :: {integer(), pos_integer()}
```

Returns a pair of integers whose ratio is exactly equal
to the original float and with a positive denominator.

#### Examples

    iex> Float.ratio(0.0)
    {0, 1}
    iex> Float.ratio(3.14)
    {7070651414971679, 2251799813685248}
    iex> Float.ratio(-3.14)
    {-7070651414971679, 2251799813685248}
    iex> Float.ratio(1.5)
    {3, 2}
    iex> Float.ratio(-1.5)
    {-3, 2}
    iex> Float.ratio(16.0)
    {16, 1}
    iex> Float.ratio(-16.0)
    {-16, 1}

### round(float, precision \\ 0)

```elixir
@spec round(float(), precision_range()) :: float()
```

Rounds a floating-point value to an arbitrary number of fractional
digits (between 0 and 15).

The rounding direction always ties to half up. The operation is
performed on the binary floating point, without a conversion to decimal.

This function only accepts floats and always returns a float. Use
`Kernel.round/1` if you want a function that accepts both floats
and integers and always returns an integer.

#### Known issues

The behavior of `round/2` for floats can be surprising. For example:

    iex> Float.round(5.5675, 3)
    5.567

One may have expected it to round to the half up 5.568. This is not a bug.
Most decimal fractions cannot be represented as a binary floating point
and therefore the number above is internally represented as 5.567499999,
which explains the behavior above. If you want exact rounding for decimals,
you must use a decimal library. The behavior above is also in accordance
to reference implementations, such as "Correctly Rounded Binary-Decimal and
Decimal-Binary Conversions" by David M. Gay.

#### Examples

    iex> Float.round(12.5)
    13.0
    iex> Float.round(5.5674, 3)
    5.567
    iex> Float.round(5.5675, 3)
    5.567
    iex> Float.round(-5.5674, 3)
    -5.567
    iex> Float.round(-5.5675)
    -6.0
    iex> Float.round(12.341444444444441, 15)
    12.341444444444441
    iex> Float.round(-0.01)
    -0.0

### to_charlist(float)

```elixir
@spec to_charlist(float()) :: charlist()
```

Returns a charlist which corresponds to the shortest text representation
of the given float.

The underlying algorithm changes depending on the Erlang/OTP version:

- For OTP \>= 24, it uses the algorithm presented in "Ryū: fast
  float-to-string conversion" in Proceedings of the SIGPLAN '2018
  Conference on Programming Language Design and Implementation.

- For OTP \< 24, it uses the algorithm presented in "Printing Floating-Point
  Numbers Quickly and Accurately" in Proceedings of the SIGPLAN '1996
  Conference on Programming Language Design and Implementation.

For a configurable representation, use `:erlang.float_to_list/2`.

Inlined by the compiler.

#### Examples

    iex> Float.to_charlist(7.0)
    ~c"7.0"

### to_string(float)

```elixir
@spec to_string(float()) :: String.t()
```

Returns a binary which corresponds to the shortest text representation
of the given float.

The underlying algorithm changes depending on the Erlang/OTP version:

- For OTP \>= 24, it uses the algorithm presented in "Ryū: fast
  float-to-string conversion" in Proceedings of the SIGPLAN '2018
  Conference on Programming Language Design and Implementation.

- For OTP \< 24, it uses the algorithm presented in "Printing Floating-Point
  Numbers Quickly and Accurately" in Proceedings of the SIGPLAN '1996
  Conference on Programming Language Design and Implementation.

For a configurable representation, use `:erlang.float_to_binary/2`.

Inlined by the compiler.

#### Examples

    iex> Float.to_string(7.0)
    "7.0"



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
