# Range 
(Elixir v1.18.0-dev)

Ranges represent a sequence of zero, one or many, ascending
or descending integers with a common difference called step.

The most common form of creating and matching on ranges is
via the [`first..last`](\`../2\`) and [`first..last//step`](\`..///3\`)
notations, auto-imported from `Kernel`:

    iex> 1 in 1..10
    true
    iex> 5 in 1..10
    true
    iex> 10 in 1..10
    true

Ranges are always inclusive in Elixir. When a step is defined,
integers will only belong to the range if they match the step:

    iex> 5 in 1..10//2
    true
    iex> 4 in 1..10//2
    false

When defining a range without a step, the step will be
defined based on the first and last position of the
range, If `last >= first`, it will be an increasing range
with a step of 1. Otherwise, it is a decreasing range.
Note, however, implicit decreasing ranges are deprecated.
Therefore, if you need a decreasing range from `3` to `1`,
prefer to write `3..1//-1` instead.

`../0` can also be used as a shortcut to create the range `0..-1//1`,
also known as the full-slice range:

    iex> ..
    0..-1//1

## Use cases

Ranges typically have two uses in Elixir: as a collection or
to represent a slice of another data structure.

### Ranges as collections

Ranges in Elixir are enumerables and therefore can be used
with the `Enum` module:

    iex> Enum.to_list(1..3)
    [1, 2, 3]
    iex> Enum.to_list(3..1//-1)
    [3, 2, 1]
    iex> Enum.to_list(1..5//2)
    [1, 3, 5]

Ranges may also have a single element:

    iex> Enum.to_list(1..1)
    [1]
    iex> Enum.to_list(1..1//2)
    [1]

Or even no elements at all:

    iex> Enum.to_list(10..0//1)
    []
    iex> Enum.to_list(0..10//-1)
    []

The full-slice range, returned by `../0`, is an empty collection:

    iex> Enum.to_list(..)
    []

### Ranges as slices

Ranges are also frequently used to slice collections.
You can slice strings or any enumerable:

    iex> String.slice("elixir", 1..4)
    "lixi"
    iex> Enum.slice([0, 1, 2, 3, 4, 5], 1..4)
    [1, 2, 3, 4]

In those cases, the first and last values of the range
are mapped to positions in the collections.

If a negative number is given, it maps to a position
from the back:

    iex> String.slice("elixir", 1..-2//1)
    "lixi"
    iex> Enum.slice([0, 1, 2, 3, 4, 5], 1..-2//1)
    [1, 2, 3, 4]

The range `0..-1//1`, returned by `../0`, returns the
collection as is, which is why it is called the full-slice
range:

    iex> String.slice("elixir", ..)
    "elixir"
    iex> Enum.slice([0, 1, 2, 3, 4, 5], ..)
    [0, 1, 2, 3, 4, 5]

## Definition

An increasing range `first..last//step` is a range from `first`
to `last` increasing by `step` where  `step` must be a positive
integer and all values `v` must be `first <= v and v <= last`.
Therefore, a range `10..0//1` is an empty range because there
is no value `v` that is `10 <= v and v <= 0`.

Similarly, a decreasing range `first..last//step` is a range
from `first` to `last` decreasing by `step` where `step` must
be a negative integer and  values `v` must be `first >= v and v >= last`.
Therefore, a range `0..10//-1` is an empty range because there
is no value `v` that is `0 >= v and v >= 10`.

## Representation

Internally, ranges are represented as structs:

    iex> range = 1..9//2
    1..9//2
    iex> first..last//step = range
    iex> first
    1
    iex> last
    9
    iex> step
    2
    iex> range.step
    2

You can access the range fields (`first`, `last`, and `step`)
directly but you should not modify nor create ranges by hand.
Instead use the proper operators or `new/2` and `new/3`.

Ranges implement the `Enumerable` protocol with memory
efficient versions of all `Enumerable` callbacks:

    iex> range = 1..10
    1..10
    iex> Enum.reduce(range, 0, fn i, acc -> i * i + acc end)
    385
    iex> Enum.count(range)
    10
    iex> Enum.member?(range, 11)
    false
    iex> Enum.member?(range, 8)
    true

Such function calls are efficient memory-wise no matter the
size of the range. The implementation of the `Enumerable`
protocol uses logic based solely on the endpoints and does
not materialize the whole list of integers.

## Types

### limit()

```elixir
@type limit() :: integer()
```



### step()

```elixir
@type step() :: pos_integer() | neg_integer()
```



### t()

```elixir
@type t() :: %Range{first: limit(), last: limit(), step: step()}
```



### t(first, last)

```elixir
@type t(first, last) :: %Range{first: first, last: last, step: step()}
```



## Functions

### disjoint?(range1, range2)
*(since 1.8.0)* 
```elixir
@spec disjoint?(t(), t()) :: boolean()
```

Checks if two ranges are disjoint.

#### Examples

    iex> Range.disjoint?(1..5, 6..9)
    true
    iex> Range.disjoint?(5..1//-1, 6..9)
    true
    iex> Range.disjoint?(1..5, 5..9)
    false
    iex> Range.disjoint?(1..5, 2..7)
    false

Steps are also considered when computing the ranges to be disjoint:

    iex> Range.disjoint?(1..10//2, 2..10//2)
    true
    
    # First element in common is 29
    iex> Range.disjoint?(1..100//14, 8..100//21)
    false
    iex> Range.disjoint?(57..-1//-14, 8..100//21)
    false
    iex> Range.disjoint?(1..100//14, 50..8//-21)
    false
    iex> Range.disjoint?(1..28//14, 8..28//21)
    true
    
    # First element in common is 14
    iex> Range.disjoint?(2..28//3, 9..28//5)
    false
    iex> Range.disjoint?(26..2//-3, 29..9//-5)
    false
    
    # Starting from the back without alignment
    iex> Range.disjoint?(27..11//-3, 30..0//-7)
    true

### new(first, last)

```elixir
@spec new(limit(), limit()) :: t()
```

Creates a new range.

If `first` is less than `last`, the range will be increasing from
`first` to `last`. If `first` is equal to `last`, the range will contain
one element, which is the number itself.

If `first` is greater than `last`, the range will be decreasing from `first`
to `last`, albeit this behavior is deprecated. Therefore, it is advised to
explicitly list the step with `new/3`.

#### Examples

    iex> Range.new(-100, 100)
    -100..100

### new(first, last, step)
*(since 1.12.0)* 
```elixir
@spec new(limit(), limit(), step()) :: t()
```

Creates a new range with `step`.

#### Examples

    iex> Range.new(-100, 100, 2)
    -100..100//2

### shift(arg, steps_to_shift)
*(since 1.14.0)* 
```elixir
@spec shift(t(), integer()) :: t()
```

Shifts a range by the given number of steps.

#### Examples

    iex> Range.shift(0..10, 1)
    1..11
    iex> Range.shift(0..10, 2)
    2..12
    
    iex> Range.shift(0..10//2, 2)
    4..14//2
    iex> Range.shift(10..0//-2, 2)
    6..-4//-2

### size(range)
*(since 1.12.0)* 
```elixir
@spec size(t()) :: non_neg_integer()
```

Returns the size of `range`.

#### Examples

    iex> Range.size(1..10)
    10
    iex> Range.size(1..10//2)
    5
    iex> Range.size(1..10//3)
    4
    iex> Range.size(1..10//-1)
    0
    
    iex> Range.size(10..1//-1)
    10
    iex> Range.size(10..1//-2)
    5
    iex> Range.size(10..1//-3)
    4
    iex> Range.size(10..1//1)
    0

### split(range, split)
*(since 1.15.0)* 
```elixir
@spec split(t(), integer()) :: {t(), t()}
```

Splits a range in two.

It returns a tuple of two elements.

If `split` is less than the number of elements in the range, the first
element in the range will have `split` entries and the second will have
all remaining entries.

If `split` is more than the number of elements in the range, the second
range in the tuple will emit zero elements.

#### Examples

Increasing ranges:

    iex> Range.split(1..5, 2)
    {1..2, 3..5}
    
    iex> Range.split(1..5//2, 2)
    {1..3//2, 5..5//2}
    
    iex> Range.split(1..5//2, 0)
    {1..-1//2, 1..5//2}
    
    iex> Range.split(1..5//2, 10)
    {1..5//2, 7..5//2}

Decreasing ranges can also be split:

    iex> Range.split(5..1//-1, 2)
    {5..4//-1, 3..1//-1}
    
    iex> Range.split(5..1//-2, 2)
    {5..3//-2, 1..1//-2}
    
    iex> Range.split(5..1//-2, 0)
    {5..7//-2, 5..1//-2}
    
    iex> Range.split(5..1//-2, 10)
    {5..1//-2, -1..1//-2}

Empty ranges preserve their property but still return empty ranges:

    iex> Range.split(2..5//-1, 2)
    {2..3//-1, 4..5//-1}
    
    iex> Range.split(2..5//-1, 10)
    {2..3//-1, 4..5//-1}
    
    iex> Range.split(5..2//1, 2)
    {5..4//1, 3..2//1}
    
    iex> Range.split(5..2//1, 10)
    {5..4//1, 3..2//1}

If the number to split is negative, it splits from the back:

    iex> Range.split(1..5, -2)
    {1..3, 4..5}
    
    iex> Range.split(5..1//-1, -2)
    {5..3//-1, 2..1//-1}

If it is negative and greater than the elements in the range,
the first element of the tuple will be an empty range:

    iex> Range.split(1..5, -10)
    {1..0//1, 1..5}
    
    iex> Range.split(5..1//-1, -10)
    {5..6//-1, 5..1//-1}

#### Properties

When a range is split, the following properties are observed.
Given `split(input)` returns `{left, right}`, we have:

    assert input.first == left.first
    assert input.last == right.last
    assert input.step == left.step
    assert input.step == right.step
    assert Range.size(input) == Range.size(left) + Range.size(right)

### to_list(arg1)
*(since 1.15.0)* 
```elixir
@spec to_list(t()) :: [integer()]
```

Converts a range to a list.

#### Examples

    iex> Range.to_list(0..5)
    [0, 1, 2, 3, 4, 5]
    iex> Range.to_list(-3..0)
    [-3, -2, -1, 0]



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
