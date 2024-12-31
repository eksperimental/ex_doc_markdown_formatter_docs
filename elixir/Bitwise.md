# Bitwise 
(Elixir v1.18.0-dev)

A set of functions that perform calculations on bits.

All bitwise functions work only on integers, otherwise an
`ArithmeticError` is raised. The functions `band/2`,
`bor/2`, `bsl/2`, and `bsr/2` also have operators,
respectively: `&&&/2`, `|||/2`, `<<</2`, and `>>>/2`.

## Guards

All bitwise functions can be used in guards:

    iex> odd? = fn
    ...>   int when Bitwise.band(int, 1) == 1 -> true
    ...>   _ -> false
    ...> end
    iex> odd?.(1)
    true

All functions in this module are inlined by the compiler.


## Guards

### left &amp;&amp;&amp; right

```elixir
@spec integer() &amp;&amp;&amp; integer() :: integer()
```

Bitwise AND operator.

Calculates the bitwise AND of its arguments.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 9 &&& 3
    1


### left &lt;&lt;&lt; right

```elixir
@spec integer() &lt;&lt;&lt; integer() :: integer()
```

Arithmetic left bitshift operator.

Calculates the result of an arithmetic left bitshift.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 <<< 2
    4
    
    iex> 1 <<< -2
    0
    
    iex> -1 <<< 2
    -4
    
    iex> -1 <<< -2
    -1


### left &gt;&gt;&gt; right

```elixir
@spec integer() &gt;&gt;&gt; integer() :: integer()
```

Arithmetic right bitshift operator.

Calculates the result of an arithmetic right bitshift.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 >>> 2
    0
    
    iex> 1 >>> -2
    4
    
    iex> -1 >>> 2
    -1
    
    iex> -1 >>> -2
    -4


### band(left, right)

```elixir
@spec band(integer(), integer()) :: integer()
```

Calculates the bitwise AND of its arguments.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> band(9, 3)
    1


### bnot(expr)

```elixir
@spec bnot(integer()) :: integer()
```

Calculates the bitwise NOT of the argument.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> bnot(2)
    -3
    
    iex> bnot(2) &&& 3
    1


### bor(left, right)

```elixir
@spec bor(integer(), integer()) :: integer()
```

Calculates the bitwise OR of its arguments.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> bor(9, 3)
    11


### bsl(left, right)

```elixir
@spec bsl(integer(), integer()) :: integer()
```

Calculates the result of an arithmetic left bitshift.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> bsl(1, 2)
    4
    
    iex> bsl(1, -2)
    0
    
    iex> bsl(-1, 2)
    -4
    
    iex> bsl(-1, -2)
    -1


### bsr(left, right)

```elixir
@spec bsr(integer(), integer()) :: integer()
```

Calculates the result of an arithmetic right bitshift.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> bsr(1, 2)
    0
    
    iex> bsr(1, -2)
    4
    
    iex> bsr(-1, 2)
    -1
    
    iex> bsr(-1, -2)
    -4


### bxor(left, right)

```elixir
@spec bxor(integer(), integer()) :: integer()
```

Calculates the bitwise XOR of its arguments.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> bxor(9, 3)
    10


### left ||| right

```elixir
@spec integer() ||| integer() :: integer()
```

Bitwise OR operator.

Calculates the bitwise OR of its arguments.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 9 ||| 3
    11




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
