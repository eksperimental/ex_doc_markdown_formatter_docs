# Duration 
(Elixir v1.18.0-dev)

Struct and functions for handling durations.

A `Duration` struct represents a collection of time scale units,
allowing for manipulation and calculation of durations.

Date and time scale units are represented as integers, allowing for
both positive and negative values.

Microseconds are represented using a tuple `{microsecond, precision}`.
This ensures compatibility with other calendar types implementing time,
such as `Time`, `DateTime`, and `NaiveDateTime`.

## Shifting

The most common use of durations in Elixir's standard library is to
"shift" the calendar types.

    iex> Date.shift(~D[2016-01-03], month: 2)
    ~D[2016-03-03]

In the example above, `Date.shift/2` automatically converts the units
into a `Duration` struct, although one can also be given directly:

    iex> Date.shift(~D[2016-01-03], Duration.new!(month: 2))
    ~D[2016-03-03]

It is important to note that shifting is not an arithmetic operation.
For example, adding `date + 1 month + 1 month` does not yield the same
result as `date + 2 months`. Let's see an example:

    iex> ~D[2016-01-31] |> Date.shift(month: 1) |> Date.shift(month: 1)
    ~D[2016-03-29]
    
    iex> ~D[2016-01-31] |> Date.shift(month: 2)
    ~D[2016-03-31]

As you can see above, the results differ, which explains why operations
with durations are called "shift" rather than "add". This happens because,
once we add one month to `2016-01-31`, we get `2016-02-29`. Then adding
one extra month gives us `2016-03-29` instead of `2016-03-31`.

In particular, when applying durations to `Calendar.ISO` types:

- larger units (such as years and months) are applied before
  smaller ones (such as weeks, hours, days, and so on)

- units are collapsed into months (`:year` and `:month`),
  seconds (`:week`, `:day`, `:hour`, `:minute`, `:second`)
  and microseconds (`:microsecond`) before they are applied

- 1 year is equivalent to 12 months, 1 week is equivalent to 7 days.
  Therefore, 4 weeks *are not* equivalent to 1 month

- in case of non-existing dates, the results are rounded down to the
  nearest valid date

As the `shift/2` functions are calendar aware, they are guaranteed to return
valid date/times, considering leap years as well as DST in applicable time zones.

## Intervals

Durations in Elixir can be combined with stream operations to build intervals.
For example, to retrieve the next three Wednesdays starting from 17th April, 2024:

    iex> ~D[2024-04-17] |> Stream.iterate(&Date.shift(&1, week: 1)) |> Enum.take(3)
    [~D[2024-04-17], ~D[2024-04-24], ~D[2024-05-01]]

However, once again, it is important to remember that shifting a duration is not
arithmetic, so you may want to use the functions in this module depending on what
you to achieve. Compare the results of both examples below:

    # Adding one month after the other
    iex> date = ~D[2016-01-31]
    iex> duration = Duration.new!(month: 1)
    iex> stream = Stream.iterate(date, fn prev_date -> Date.shift(prev_date, duration) end)
    iex> Enum.take(stream, 3)
    [~D[2016-01-31], ~D[2016-02-29], ~D[2016-03-29]]
    
    # Multiplying durations by an index
    iex> date = ~D[2016-01-31]
    iex> duration = Duration.new!(month: 1)
    iex> stream = Stream.from_index(fn i -> Date.shift(date, Duration.multiply(duration, i)) end)
    iex> Enum.take(stream, 3)
    [~D[2016-01-31], ~D[2016-02-29], ~D[2016-03-31]]

The second example consistently points to the last day of the month,
as it performs operations on the duration, rather than shifting date
after date.

## Types

### duration()
*(since 1.17.0)* 
```elixir
@type duration() :: t() | [unit_pair()]
```

The duration type specifies a `%Duration{}` struct or a keyword list of valid duration unit pairs.

### t()
*(since 1.17.0)* 
```elixir
@type t() :: %Duration{
  day: integer(),
  hour: integer(),
  microsecond: {integer(), 0..6},
  minute: integer(),
  month: integer(),
  second: integer(),
  week: integer(),
  year: integer()
}
```

The duration struct type.

### unit_pair()
*(since 1.17.0)* 
```elixir
@type unit_pair() ::
  {:year, integer()}
  | {:month, integer()}
  | {:week, integer()}
  | {:day, integer()}
  | {:hour, integer()}
  | {:minute, integer()}
  | {:second, integer()}
  | {:microsecond, {integer(), 0..6}}
```

The unit pair type specifies a pair of a valid duration unit key and value.

## Functions

### add(d1, d2)
*(since 1.17.0)* 
```elixir
@spec add(t(), t()) :: t()
```

Adds units of given durations `d1` and `d2`.

Respects the the highest microsecond precision of the two.

#### Examples

    iex> Duration.add(Duration.new!(week: 2, day: 1), Duration.new!(day: 2))
    %Duration{week: 2, day: 3}
    iex> Duration.add(Duration.new!(microsecond: {400, 3}), Duration.new!(microsecond: {600, 6}))
    %Duration{microsecond: {1000, 6}}

### from_iso8601(string)
*(since 1.17.0)* 
```elixir
@spec from_iso8601(String.t()) :: {:ok, t()} | {:error, atom()}
```

Parses an [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Durations) formatted duration string to a `Duration` struct.

Duration strings, as well as individual units, may be prefixed with plus/minus signs so that:

- `-PT6H3M` parses as `%Duration{hour: -6, minute: -3}`
- `-PT6H-3M` parses as `%Duration{hour: -6, minute: 3}`
- `+PT6H3M` parses as `%Duration{hour: 6, minute: 3}`
- `+PT6H-3M` parses as `%Duration{hour: 6, minute: -3}`

Duration designators must be provided in order of magnitude: `P[n]Y[n]M[n]W[n]DT[n]H[n]M[n]S`.

Only seconds may be specified with a decimal fraction, using either a comma or a full stop: `P1DT4,5S`.

#### Examples

    iex> Duration.from_iso8601("P1Y2M3DT4H5M6S")
    {:ok, %Duration{year: 1, month: 2, day: 3, hour: 4, minute: 5, second: 6}}
    iex> Duration.from_iso8601("P3Y-2MT3H")
    {:ok, %Duration{year: 3, month: -2, hour: 3}}
    iex> Duration.from_iso8601("-PT10H-30M")
    {:ok, %Duration{hour: -10, minute: 30}}
    iex> Duration.from_iso8601("PT4.650S")
    {:ok, %Duration{second: 4, microsecond: {650000, 3}}}

### from_iso8601!(string)
*(since 1.17.0)* 
```elixir
@spec from_iso8601!(String.t()) :: t()
```

Same as `from_iso8601/1` but raises an `ArgumentError`.

#### Examples

    iex> Duration.from_iso8601!("P1Y2M3DT4H5M6S")
    %Duration{year: 1, month: 2, day: 3, hour: 4, minute: 5, second: 6}
    iex> Duration.from_iso8601!("P10D")
    %Duration{day: 10}

### multiply(duration, integer)
*(since 1.17.0)* 
```elixir
@spec multiply(t(), integer()) :: t()
```

Multiplies `duration` units by given `integer`.

#### Examples

    iex> Duration.multiply(Duration.new!(day: 1, minute: 15, second: -10), 3)
    %Duration{day: 3, minute: 45, second: -30}
    iex> Duration.multiply(Duration.new!(microsecond: {200, 4}), 3)
    %Duration{microsecond: {600, 4}}

### negate(duration)
*(since 1.17.0)* 
```elixir
@spec negate(t()) :: t()
```

Negates `duration` units.

#### Examples

    iex> Duration.negate(Duration.new!(day: 1, minute: 15, second: -10))
    %Duration{day: -1, minute: -15, second: 10}
    iex> Duration.negate(Duration.new!(microsecond: {500000, 4}))
    %Duration{microsecond: {-500000, 4}}

### new!(duration)
*(since 1.17.0)* 
```elixir
@spec new!(duration()) :: t()
```

Creates a new `Duration` struct from given `unit_pairs`.

Raises an `ArgumentError` when called with invalid unit pairs.

#### Examples

    iex> Duration.new!(year: 1, week: 3, hour: 4, second: 1)
    %Duration{year: 1, week: 3, hour: 4, second: 1}
    iex> Duration.new!(second: 1, microsecond: {1000, 6})
    %Duration{second: 1, microsecond: {1000, 6}}
    iex> Duration.new!(month: 2)
    %Duration{month: 2}

### subtract(d1, d2)
*(since 1.17.0)* 
```elixir
@spec subtract(t(), t()) :: t()
```

Subtracts units of given durations `d1` and `d2`.

Respects the the highest microsecond precision of the two.

#### Examples

    iex> Duration.subtract(Duration.new!(week: 2, day: 1), Duration.new!(day: 2))
    %Duration{week: 2, day: -1}
    iex> Duration.subtract(Duration.new!(microsecond: {400, 6}), Duration.new!(microsecond: {600, 3}))
    %Duration{microsecond: {-200, 6}}

### to_iso8601(duration)
*(since 1.17.0)* 
```elixir
@spec to_iso8601(t()) :: String.t()
```

Converts the given `duration` to an [ISO 8601-2:2019](https://en.wikipedia.org/wiki/ISO_8601) formatted string.

Note this function implements the *extension* of ISO 8601:2019. This extensions allows weeks to
appear between months and days: `P3M3W3D`, making it fully compatible with any `Duration` struct.

#### Examples

    iex> Duration.to_iso8601(Duration.new!(year: 3))
    "P3Y"
    iex> Duration.to_iso8601(Duration.new!(day: 40, hour: 12, minute: 42, second: 12))
    "P40DT12H42M12S"
    iex> Duration.to_iso8601(Duration.new!(second: 30))
    "PT30S"
    
    iex> Duration.to_iso8601(Duration.new!([]))
    "PT0S"
    
    iex> Duration.to_iso8601(Duration.new!(second: 1, microsecond: {2_200, 3}))
    "PT1.002S"
    iex> Duration.to_iso8601(Duration.new!(second: 1, microsecond: {-1_200_000, 4}))
    "PT-0.2000S"



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
