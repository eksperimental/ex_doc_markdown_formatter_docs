# NaiveDateTime 
(Elixir v1.18.0-dev)

A NaiveDateTime struct (without a time zone) and functions.

The NaiveDateTime struct contains the fields year, month, day, hour,
minute, second, microsecond and calendar. New naive datetimes can be
built with the `new/2` and `new/8` functions or using the
`~N` (see `sigil_N/2`) sigil:

    iex> ~N[2000-01-01 23:00:07]
    ~N[2000-01-01 23:00:07]

The date and time fields in the struct can be accessed directly:

    iex> naive = ~N[2000-01-01 23:00:07]
    iex> naive.year
    2000
    iex> naive.second
    7

We call them "naive" because this datetime representation does not
have a time zone. This means the datetime may not actually exist in
certain areas in the world even though it is valid.

For example, when daylight saving changes are applied by a region,
the clock typically moves forward or backward by one hour. This means
certain datetimes never occur or may occur more than once. Since
`NaiveDateTime` is not validated against a time zone, such errors
would go unnoticed.

Developers should avoid creating the NaiveDateTime structs directly
and instead, rely on the functions provided by this module as well
as the ones in third-party calendar libraries.

## Comparing naive date times

Comparisons in Elixir using `==/2`, `>/2`, `</2` and similar are structural
and based on the `NaiveDateTime` struct fields. For proper comparison
between naive datetimes, use the `compare/2` function. The existence of the
`compare/2` function in this module also allows using `Enum.min/2` and
`Enum.max/2` functions to get the minimum and maximum naive datetime of an
`Enum`. For example:

    iex> Enum.min([~N[2020-01-01 23:00:07], ~N[2000-01-01 23:00:07]], NaiveDateTime)
    ~N[2000-01-01 23:00:07]

## Using epochs

The `add/3` and `diff/3` functions can be used for computing date
times or retrieving the number of seconds between instants.
For example, if there is an interest in computing the number of
seconds from the Unix epoch (1970-01-01 00:00:00):

    iex> NaiveDateTime.diff(~N[2010-04-17 14:00:00], ~N[1970-01-01 00:00:00])
    1271512800
    
    iex> NaiveDateTime.add(~N[1970-01-01 00:00:00], 1_271_512_800)
    ~N[2010-04-17 14:00:00]

Those functions are optimized to deal with common epochs, such
as the Unix Epoch above or the Gregorian Epoch (0000-01-01 00:00:00).


## Types

### t()

```elixir
@type t() :: %NaiveDateTime{
  calendar: Calendar.calendar(),
  day: Calendar.day(),
  hour: Calendar.hour(),
  microsecond: Calendar.microsecond(),
  minute: Calendar.minute(),
  month: Calendar.month(),
  second: Calendar.second(),
  year: Calendar.year()
}
```



## Functions

### add(naive_datetime, amount_to_add, unit \\ :second)
*(since 1.4.0)* 
```elixir
@spec add(
  Calendar.naive_datetime(),
  integer(),
  :day | :hour | :minute | System.time_unit()
) :: t()
```

Adds a specified amount of time to a `NaiveDateTime`.

Accepts an `amount_to_add` in any `unit`. `unit` can be `:day`,
`:hour`, `:minute`, `:second` or any subsecond precision from
`t:System.time_unit/0`. It defaults to `:second`. Negative values
will move backwards in time.

This function always consider the unit to be computed according
to the `Calendar.ISO`.

#### Examples

It uses seconds by default:

    # adds seconds by default
    iex> NaiveDateTime.add(~N[2014-10-02 00:29:10], 2)
    ~N[2014-10-02 00:29:12]
    
    # accepts negative offsets
    iex> NaiveDateTime.add(~N[2014-10-02 00:29:10], -2)
    ~N[2014-10-02 00:29:08]

It can also work with subsecond precisions:

    iex> NaiveDateTime.add(~N[2014-10-02 00:29:10], 2_000, :millisecond)
    ~N[2014-10-02 00:29:12.000]

As well as days/hours/minutes:

    iex> NaiveDateTime.add(~N[2015-02-28 00:29:10], 2, :day)
    ~N[2015-03-02 00:29:10]
    iex> NaiveDateTime.add(~N[2015-02-28 00:29:10], 36, :hour)
    ~N[2015-03-01 12:29:10]
    iex> NaiveDateTime.add(~N[2015-02-28 00:29:10], 60, :minute)
    ~N[2015-02-28 01:29:10]

This operation merges the precision of the naive date time with the given unit:

    iex> result = NaiveDateTime.add(~N[2014-10-02 00:29:10], 21, :millisecond)
    ~N[2014-10-02 00:29:10.021]
    iex> result.microsecond
    {21000, 3}

Operations on top of gregorian seconds or the Unix epoch are optimized:

    # from Gregorian seconds
    iex> NaiveDateTime.add(~N[0000-01-01 00:00:00], 63_579_428_950)
    ~N[2014-10-02 00:29:10]

Passing a `DateTime` automatically converts it to `NaiveDateTime`,
discarding the time zone information:

    iex> dt = %DateTime{year: 2000, month: 2, day: 29, zone_abbr: "CET",
    ...>                hour: 23, minute: 0, second: 7, microsecond: {0, 0},
    ...>                utc_offset: 3600, std_offset: 0, time_zone: "Europe/Warsaw"}
    iex> NaiveDateTime.add(dt, 21, :second)
    ~N[2000-02-29 23:00:28]

To shift a naive datetime by a `Duration` and according to its underlying calendar, use `NaiveDateTime.shift/2`.


### after?(naive_datetime1, naive_datetime2)
*(since 1.15.0)* 
```elixir
@spec after?(Calendar.naive_datetime(), Calendar.naive_datetime()) :: boolean()
```

Returns `true` if the first `NaiveDateTime` is strictly later than the second.

#### Examples

    iex> NaiveDateTime.after?(~N[2022-02-02 11:00:00], ~N[2021-01-01 11:00:00])
    true
    iex> NaiveDateTime.after?(~N[2021-01-01 11:00:00], ~N[2021-01-01 11:00:00])
    false
    iex> NaiveDateTime.after?(~N[2021-01-01 11:00:00], ~N[2022-02-02 11:00:00])
    false


### before?(naive_datetime1, naive_datetime2)
*(since 1.15.0)* 
```elixir
@spec before?(Calendar.naive_datetime(), Calendar.naive_datetime()) :: boolean()
```

Returns `true` if the first `NaiveDateTime` is strictly earlier than the second.

#### Examples

    iex> NaiveDateTime.before?(~N[2021-01-01 11:00:00], ~N[2022-02-02 11:00:00])
    true
    iex> NaiveDateTime.before?(~N[2021-01-01 11:00:00], ~N[2021-01-01 11:00:00])
    false
    iex> NaiveDateTime.before?(~N[2022-02-02 11:00:00], ~N[2021-01-01 11:00:00])
    false


### beginning_of_day(naive_datetime)
*(since 1.15.0)* 
```elixir
@spec beginning_of_day(Calendar.naive_datetime()) :: t()
```

Calculates a `NaiveDateTime` that is the first moment for the given `NaiveDateTime`.

To calculate the beginning of day of a `DateTime`, call this function, then convert back to a `DateTime`:

    datetime
    |> NaiveDateTime.beginning_of_day()
    |> DateTime.from_naive(datetime.time_zone)

Note that the beginning of the day may not exist or be ambiguous
in a given timezone, so you must handle those cases accordingly.

#### Examples

    iex> NaiveDateTime.beginning_of_day(~N[2000-01-01 23:00:07.123456])
    ~N[2000-01-01 00:00:00.000000]


### compare(naive_datetime1, naive_datetime2)
*(since 1.4.0)* 
```elixir
@spec compare(Calendar.naive_datetime(), Calendar.naive_datetime()) :: :lt | :eq | :gt
```

Compares two `NaiveDateTime` structs.

Returns `:gt` if first is later than the second
and `:lt` for vice versa. If the two NaiveDateTime
are equal `:eq` is returned.

#### Examples

    iex> NaiveDateTime.compare(~N[2016-04-16 13:30:15], ~N[2016-04-28 16:19:25])
    :lt
    iex> NaiveDateTime.compare(~N[2016-04-16 13:30:15.1], ~N[2016-04-16 13:30:15.01])
    :gt

This function can also be used to compare a DateTime without
the time zone information:

    iex> dt = %DateTime{year: 2000, month: 2, day: 29, zone_abbr: "CET",
    ...>                hour: 23, minute: 0, second: 7, microsecond: {0, 0},
    ...>                utc_offset: 3600, std_offset: 0, time_zone: "Europe/Warsaw"}
    iex> NaiveDateTime.compare(dt, ~N[2000-02-29 23:00:07])
    :eq
    iex> NaiveDateTime.compare(dt, ~N[2000-01-29 23:00:07])
    :gt
    iex> NaiveDateTime.compare(dt, ~N[2000-03-29 23:00:07])
    :lt


### convert(naive_datetime, calendar)
*(since 1.5.0)* 
```elixir
@spec convert(Calendar.naive_datetime(), Calendar.calendar()) ::
  {:ok, t()} | {:error, :incompatible_calendars}
```

Converts the given `naive_datetime` from one calendar to another.

If it is not possible to convert unambiguously between the calendars
(see `Calendar.compatible_calendars?/2`), an `{:error, :incompatible_calendars}` tuple
is returned.

#### Examples

Imagine someone implements `Calendar.Holocene`, a calendar based on the
Gregorian calendar that adds exactly 10,000 years to the current Gregorian
year:

    iex> NaiveDateTime.convert(~N[2000-01-01 13:30:15], Calendar.Holocene)
    {:ok, %NaiveDateTime{calendar: Calendar.Holocene, year: 12000, month: 1, day: 1,
                         hour: 13, minute: 30, second: 15, microsecond: {0, 0}}}


### convert!(naive_datetime, calendar)
*(since 1.5.0)* 
```elixir
@spec convert!(Calendar.naive_datetime(), Calendar.calendar()) :: t()
```

Converts the given `naive_datetime` from one calendar to another.

If it is not possible to convert unambiguously between the calendars
(see `Calendar.compatible_calendars?/2`), an ArgumentError is raised.

#### Examples

Imagine someone implements `Calendar.Holocene`, a calendar based on the
Gregorian calendar that adds exactly 10,000 years to the current Gregorian
year:

    iex> NaiveDateTime.convert!(~N[2000-01-01 13:30:15], Calendar.Holocene)
    %NaiveDateTime{calendar: Calendar.Holocene, year: 12000, month: 1, day: 1,
                   hour: 13, minute: 30, second: 15, microsecond: {0, 0}}


### diff(naive_datetime1, naive_datetime2, unit \\ :second)
*(since 1.4.0)* 
```elixir
@spec diff(
  Calendar.naive_datetime(),
  Calendar.naive_datetime(),
  :day | :hour | :minute | System.time_unit()
) :: integer()
```

Subtracts `naive_datetime2` from `naive_datetime1`.

The answer can be returned in any `:day`, `:hour`, `:minute`, or any `unit`
available from `t:System.time_unit/0`. The unit is measured according to
`Calendar.ISO` and defaults to `:second`.

Fractional results are not supported and are truncated.

#### Examples

    iex> NaiveDateTime.diff(~N[2014-10-02 00:29:12], ~N[2014-10-02 00:29:10])
    2
    iex> NaiveDateTime.diff(~N[2014-10-02 00:29:12], ~N[2014-10-02 00:29:10], :microsecond)
    2_000_000
    
    iex> NaiveDateTime.diff(~N[2014-10-02 00:29:10.042], ~N[2014-10-02 00:29:10.021])
    0
    iex> NaiveDateTime.diff(~N[2014-10-02 00:29:10.042], ~N[2014-10-02 00:29:10.021], :millisecond)
    21
    
    iex> NaiveDateTime.diff(~N[2014-10-02 00:29:10], ~N[2014-10-02 00:29:12])
    -2
    iex> NaiveDateTime.diff(~N[-0001-10-02 00:29:10], ~N[-0001-10-02 00:29:12])
    -2

It can also compute the difference in days, hours, or minutes:

    iex> NaiveDateTime.diff(~N[2014-10-10 00:29:10], ~N[2014-10-02 00:29:10], :day)
    8
    iex> NaiveDateTime.diff(~N[2014-10-02 12:29:10], ~N[2014-10-02 00:29:10], :hour)
    12
    iex> NaiveDateTime.diff(~N[2014-10-02 00:39:10], ~N[2014-10-02 00:29:10], :minute)
    10

But it also rounds incomplete days to zero:

    iex> NaiveDateTime.diff(~N[2014-10-10 00:29:09], ~N[2014-10-02 00:29:10], :day)
    7


### end_of_day(naive_datetime)
*(since 1.15.0)* 
```elixir
@spec end_of_day(Calendar.naive_datetime()) :: t()
```

Calculates a `NaiveDateTime` that is the last moment for the given `NaiveDateTime`.

To calculate the end of day of a `DateTime`, call this function, then convert back to a `DateTime`:

    datetime
    |> NaiveDateTime.end_of_day()
    |> DateTime.from_naive(datetime.time_zone)

Note that the end of the day may not exist or be ambiguous
in a given timezone, so you must handle those cases accordingly.

#### Examples

    iex> NaiveDateTime.end_of_day(~N[2000-01-01 23:00:07.123456])
    ~N[2000-01-01 23:59:59.999999]


### from_erl(tuple, microsecond \\ {0, 0}, calendar \\ Calendar.ISO)

```elixir
@spec from_erl(
  :calendar.datetime(),
  Calendar.microsecond() | non_neg_integer(),
  Calendar.calendar()
) :: {:ok, t()} | {:error, atom()}
```

Converts an Erlang datetime tuple to a `NaiveDateTime` struct.

Attempting to convert an invalid ISO calendar date will produce an error tuple.

#### Examples

    iex> NaiveDateTime.from_erl({{2000, 1, 1}, {13, 30, 15}})
    {:ok, ~N[2000-01-01 13:30:15]}
    iex> NaiveDateTime.from_erl({{2000, 1, 1}, {13, 30, 15}}, 5000)
    {:ok, ~N[2000-01-01 13:30:15.005000]}
    iex> NaiveDateTime.from_erl({{2000, 1, 1}, {13, 30, 15}}, {5000, 3})
    {:ok, ~N[2000-01-01 13:30:15.005]}
    iex> NaiveDateTime.from_erl({{2000, 13, 1}, {13, 30, 15}})
    {:error, :invalid_date}
    iex> NaiveDateTime.from_erl({{2000, 13, 1}, {13, 30, 15}})
    {:error, :invalid_date}


### from_erl!(tuple, microsecond \\ {0, 0}, calendar \\ Calendar.ISO)

```elixir
@spec from_erl!(
  :calendar.datetime(),
  Calendar.microsecond() | non_neg_integer(),
  Calendar.calendar()
) :: t()
```

Converts an Erlang datetime tuple to a `NaiveDateTime` struct.

Raises if the datetime is invalid.
Attempting to convert an invalid ISO calendar date will produce an error tuple.

#### Examples

    iex> NaiveDateTime.from_erl!({{2000, 1, 1}, {13, 30, 15}})
    ~N[2000-01-01 13:30:15]
    iex> NaiveDateTime.from_erl!({{2000, 1, 1}, {13, 30, 15}}, 5000)
    ~N[2000-01-01 13:30:15.005000]
    iex> NaiveDateTime.from_erl!({{2000, 1, 1}, {13, 30, 15}}, {5000, 3})
    ~N[2000-01-01 13:30:15.005]
    iex> NaiveDateTime.from_erl!({{2000, 13, 1}, {13, 30, 15}})
    ** (ArgumentError) cannot convert {{2000, 13, 1}, {13, 30, 15}} to naive datetime, reason: :invalid_date


### from_gregorian_seconds(seconds, microsecond_precision \\ {0, 0}, calendar \\ Calendar.ISO)
*(since 1.11.0)* 
```elixir
@spec from_gregorian_seconds(integer(), Calendar.microsecond(), Calendar.calendar()) ::
  t()
```

Converts a number of gregorian seconds to a `NaiveDateTime` struct.

#### Examples

    iex> NaiveDateTime.from_gregorian_seconds(1)
    ~N[0000-01-01 00:00:01]
    iex> NaiveDateTime.from_gregorian_seconds(63_755_511_991, {5000, 3})
    ~N[2020-05-01 00:26:31.005]
    iex> NaiveDateTime.from_gregorian_seconds(-1)
    ~N[-0001-12-31 23:59:59]


### from_iso8601(string, calendar \\ Calendar.ISO)

```elixir
@spec from_iso8601(String.t(), Calendar.calendar()) :: {:ok, t()} | {:error, atom()}
```

Parses the extended "Date and time of day" format described by
[ISO 8601:2019](https://en.wikipedia.org/wiki/ISO_8601).

Time zone offset may be included in the string but they will be
simply discarded as such information is not included in naive date
times.

As specified in the standard, the separator "T" may be omitted if
desired as there is no ambiguity within this function.

Note leap seconds are not supported by the built-in Calendar.ISO.

#### Examples

    iex> NaiveDateTime.from_iso8601("2015-01-23 23:50:07")
    {:ok, ~N[2015-01-23 23:50:07]}
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07")
    {:ok, ~N[2015-01-23 23:50:07]}
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07Z")
    {:ok, ~N[2015-01-23 23:50:07]}
    
    iex> NaiveDateTime.from_iso8601("2015-01-23 23:50:07.0")
    {:ok, ~N[2015-01-23 23:50:07.0]}
    iex> NaiveDateTime.from_iso8601("2015-01-23 23:50:07,0123456")
    {:ok, ~N[2015-01-23 23:50:07.012345]}
    iex> NaiveDateTime.from_iso8601("2015-01-23 23:50:07.0123456")
    {:ok, ~N[2015-01-23 23:50:07.012345]}
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07.123Z")
    {:ok, ~N[2015-01-23 23:50:07.123]}
    
    iex> NaiveDateTime.from_iso8601("2015-01-23P23:50:07")
    {:error, :invalid_format}
    iex> NaiveDateTime.from_iso8601("2015:01:23 23-50-07")
    {:error, :invalid_format}
    iex> NaiveDateTime.from_iso8601("2015-01-23 23:50:07A")
    {:error, :invalid_format}
    iex> NaiveDateTime.from_iso8601("2015-01-23 23:50:61")
    {:error, :invalid_time}
    iex> NaiveDateTime.from_iso8601("2015-01-32 23:50:07")
    {:error, :invalid_date}
    
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07.123+02:30")
    {:ok, ~N[2015-01-23 23:50:07.123]}
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07.123+00:00")
    {:ok, ~N[2015-01-23 23:50:07.123]}
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07.123-02:30")
    {:ok, ~N[2015-01-23 23:50:07.123]}
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07.123-00:00")
    {:error, :invalid_format}
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07.123-00:60")
    {:error, :invalid_format}
    iex> NaiveDateTime.from_iso8601("2015-01-23T23:50:07.123-24:00")
    {:error, :invalid_format}


### from_iso8601!(string, calendar \\ Calendar.ISO)

```elixir
@spec from_iso8601!(String.t(), Calendar.calendar()) :: t()
```

Parses the extended "Date and time of day" format described by
[ISO 8601:2019](https://en.wikipedia.org/wiki/ISO_8601).

Raises if the format is invalid.

#### Examples

    iex> NaiveDateTime.from_iso8601!("2015-01-23T23:50:07.123Z")
    ~N[2015-01-23 23:50:07.123]
    iex> NaiveDateTime.from_iso8601!("2015-01-23T23:50:07,123Z")
    ~N[2015-01-23 23:50:07.123]
    iex> NaiveDateTime.from_iso8601!("2015-01-23P23:50:07")
    ** (ArgumentError) cannot parse "2015-01-23P23:50:07" as naive datetime, reason: :invalid_format


### local_now(calendar \\ Calendar.ISO)
*(since 1.10.0)* 
```elixir
@spec local_now(Calendar.calendar()) :: t()
```

Returns the "local time" for the machine the Elixir program is running on.

WARNING: This function can cause insidious bugs. It depends on the time zone
configuration at run time. This can changed and be set to a time zone that has
daylight saving jumps (spring forward or fall back).

This function can be used to display what the time is right now for the time
zone configuration that the machine happens to have. An example would be a
desktop program displaying a clock to the user. For any other uses it is
probably a bad idea to use this function.

For most cases, use `DateTime.now/2` or `DateTime.utc_now/1` instead.

Does not include fractional seconds.

#### Examples

    iex> naive_datetime = NaiveDateTime.local_now()
    iex> naive_datetime.year >= 2019
    true


### new(date, time)

```elixir
@spec new(Date.t(), Time.t()) :: {:ok, t()}
```

Builds a naive datetime from date and time structs.

#### Examples

    iex> NaiveDateTime.new(~D[2010-01-13], ~T[23:00:07.005])
    {:ok, ~N[2010-01-13 23:00:07.005]}


### new(year, month, day, hour, minute, second, microsecond \\ {0, 0}, calendar \\ Calendar.ISO)

```elixir
@spec new(
  Calendar.year(),
  Calendar.month(),
  Calendar.day(),
  Calendar.hour(),
  Calendar.minute(),
  Calendar.second(),
  Calendar.microsecond() | non_neg_integer(),
  Calendar.calendar()
) :: {:ok, t()} | {:error, atom()}
```

Builds a new ISO naive datetime.

Expects all values to be integers. Returns `{:ok, naive_datetime}`
if each entry fits its appropriate range, returns `{:error, reason}`
otherwise.

#### Examples

    iex> NaiveDateTime.new(2000, 1, 1, 0, 0, 0)
    {:ok, ~N[2000-01-01 00:00:00]}
    iex> NaiveDateTime.new(2000, 13, 1, 0, 0, 0)
    {:error, :invalid_date}
    iex> NaiveDateTime.new(2000, 2, 29, 0, 0, 0)
    {:ok, ~N[2000-02-29 00:00:00]}
    iex> NaiveDateTime.new(2000, 2, 30, 0, 0, 0)
    {:error, :invalid_date}
    iex> NaiveDateTime.new(2001, 2, 29, 0, 0, 0)
    {:error, :invalid_date}
    
    iex> NaiveDateTime.new(2000, 1, 1, 23, 59, 59, {0, 1})
    {:ok, ~N[2000-01-01 23:59:59.0]}
    iex> NaiveDateTime.new(2000, 1, 1, 23, 59, 59, 999_999)
    {:ok, ~N[2000-01-01 23:59:59.999999]}
    iex> NaiveDateTime.new(2000, 1, 1, 24, 59, 59, 999_999)
    {:error, :invalid_time}
    iex> NaiveDateTime.new(2000, 1, 1, 23, 60, 59, 999_999)
    {:error, :invalid_time}
    iex> NaiveDateTime.new(2000, 1, 1, 23, 59, 60, 999_999)
    {:error, :invalid_time}
    iex> NaiveDateTime.new(2000, 1, 1, 23, 59, 59, 1_000_000)
    {:error, :invalid_time}
    
    iex> NaiveDateTime.new(2000, 1, 1, 23, 59, 59, {0, 1}, Calendar.ISO)
    {:ok, ~N[2000-01-01 23:59:59.0]}


### new!(date, time)
*(since 1.11.0)* 
```elixir
@spec new!(Date.t(), Time.t()) :: t()
```

Builds a naive datetime from date and time structs.

#### Examples

    iex> NaiveDateTime.new!(~D[2010-01-13], ~T[23:00:07.005])
    ~N[2010-01-13 23:00:07.005]


### new!(year, month, day, hour, minute, second, microsecond \\ {0, 0}, calendar \\ Calendar.ISO)
*(since 1.11.0)* 
```elixir
@spec new!(
  Calendar.year(),
  Calendar.month(),
  Calendar.day(),
  Calendar.hour(),
  Calendar.minute(),
  Calendar.second(),
  Calendar.microsecond() | non_neg_integer(),
  Calendar.calendar()
) :: t()
```

Builds a new ISO naive datetime.

Expects all values to be integers. Returns `naive_datetime`
if each entry fits its appropriate range, raises if
time or date is invalid.

#### Examples

    iex> NaiveDateTime.new!(2000, 1, 1, 0, 0, 0)
    ~N[2000-01-01 00:00:00]
    iex> NaiveDateTime.new!(2000, 2, 29, 0, 0, 0)
    ~N[2000-02-29 00:00:00]
    iex> NaiveDateTime.new!(2000, 1, 1, 23, 59, 59, {0, 1})
    ~N[2000-01-01 23:59:59.0]
    iex> NaiveDateTime.new!(2000, 1, 1, 23, 59, 59, 999_999)
    ~N[2000-01-01 23:59:59.999999]
    iex> NaiveDateTime.new!(2000, 1, 1, 23, 59, 59, {0, 1}, Calendar.ISO)
    ~N[2000-01-01 23:59:59.0]
    iex> NaiveDateTime.new!(2000, 1, 1, 24, 59, 59, 999_999)
    ** (ArgumentError) cannot build naive datetime, reason: :invalid_time


### shift(naive_datetime, duration)
*(since 1.17.0)* 
```elixir
@spec shift(Calendar.naive_datetime(), Duration.duration()) :: t()
```

Shifts given `naive_datetime` by `duration` according to its calendar.

Allowed units are: `:year`, `:month`, `:week`, `:day`, `:hour`, `:minute`, `:second`, `:microsecond`.

When using the default ISO calendar, durations are collapsed and
applied in the order of months, then seconds and microseconds:

- when shifting by 1 year and 2 months the date is actually shifted by 14 months
- weeks, days and smaller units are collapsed into seconds and microseconds

When shifting by month, days are rounded down to the nearest valid date.

#### Examples

    iex> NaiveDateTime.shift(~N[2016-01-31 00:00:00], month: 1)
    ~N[2016-02-29 00:00:00]
    iex> NaiveDateTime.shift(~N[2016-01-31 00:00:00], year: 4, day: 1)
    ~N[2020-02-01 00:00:00]
    iex> NaiveDateTime.shift(~N[2016-01-31 00:00:00], year: -2, day: 1)
    ~N[2014-02-01 00:00:00]
    iex> NaiveDateTime.shift(~N[2016-01-31 00:00:00], second: 45)
    ~N[2016-01-31 00:00:45]
    iex> NaiveDateTime.shift(~N[2016-01-31 00:00:00], microsecond: {100, 6})
    ~N[2016-01-31 00:00:00.000100]
    
    # leap years
    iex> NaiveDateTime.shift(~N[2024-02-29 00:00:00], year: 1)
    ~N[2025-02-28 00:00:00]
    iex> NaiveDateTime.shift(~N[2024-02-29 00:00:00], year: 4)
    ~N[2028-02-29 00:00:00]
    
    # rounding down
    iex> NaiveDateTime.shift(~N[2015-01-31 00:00:00], month: 1)
    ~N[2015-02-28 00:00:00]


### to_date(map)

```elixir
@spec to_date(Calendar.naive_datetime()) :: Date.t()
```

Converts a `NaiveDateTime` into a `Date`.

Because `Date` does not hold time information,
data will be lost during the conversion.

#### Examples

    iex> NaiveDateTime.to_date(~N[2002-01-13 23:00:07])
    ~D[2002-01-13]


### to_erl(naive_datetime)

```elixir
@spec to_erl(Calendar.naive_datetime()) :: :calendar.datetime()
```

Converts a `NaiveDateTime` struct to an Erlang datetime tuple.

Only supports converting naive datetimes which are in the ISO calendar,
attempting to convert naive datetimes from other calendars will raise.

WARNING: Loss of precision may occur, as Erlang time tuples only store
hour/minute/second.

#### Examples

    iex> NaiveDateTime.to_erl(~N[2000-01-01 13:30:15])
    {{2000, 1, 1}, {13, 30, 15}}

This function can also be used to convert a DateTime to an Erlang
datetime tuple without the time zone information:

    iex> dt = %DateTime{year: 2000, month: 2, day: 29, zone_abbr: "CET",
    ...>                hour: 23, minute: 0, second: 7, microsecond: {0, 0},
    ...>                utc_offset: 3600, std_offset: 0, time_zone: "Europe/Warsaw"}
    iex> NaiveDateTime.to_erl(dt)
    {{2000, 2, 29}, {23, 00, 07}}


### to_gregorian_seconds(map)
*(since 1.11.0)* 
```elixir
@spec to_gregorian_seconds(Calendar.naive_datetime()) ::
  {integer(), non_neg_integer()}
```

Converts a `NaiveDateTime` struct to a number of gregorian seconds and microseconds.

#### Examples

    iex> NaiveDateTime.to_gregorian_seconds(~N[0000-01-01 00:00:01])
    {1, 0}
    iex> NaiveDateTime.to_gregorian_seconds(~N[2020-05-01 00:26:31.005])
    {63_755_511_991, 5000}


### to_iso8601(naive_datetime, format \\ :extended)

```elixir
@spec to_iso8601(Calendar.naive_datetime(), :basic | :extended) :: String.t()
```

Converts the given naive datetime to
[ISO 8601:2019](https://en.wikipedia.org/wiki/ISO_8601).

By default, `NaiveDateTime.to_iso8601/2` returns naive datetimes formatted in the "extended"
format, for human readability. It also supports the "basic" format through passing the `:basic` option.

Only supports converting naive datetimes which are in the ISO calendar,
attempting to convert naive datetimes from other calendars will raise.

#### Examples

    iex> NaiveDateTime.to_iso8601(~N[2000-02-28 23:00:13])
    "2000-02-28T23:00:13"
    
    iex> NaiveDateTime.to_iso8601(~N[2000-02-28 23:00:13.001])
    "2000-02-28T23:00:13.001"
    
    iex> NaiveDateTime.to_iso8601(~N[2000-02-28 23:00:13.001], :basic)
    "20000228T230013.001"

This function can also be used to convert a DateTime to ISO 8601 without
the time zone information:

    iex> dt = %DateTime{year: 2000, month: 2, day: 29, zone_abbr: "CET",
    ...>                hour: 23, minute: 0, second: 7, microsecond: {0, 0},
    ...>                utc_offset: 3600, std_offset: 0, time_zone: "Europe/Warsaw"}
    iex> NaiveDateTime.to_iso8601(dt)
    "2000-02-29T23:00:07"


### to_string(naive_datetime)

```elixir
@spec to_string(Calendar.naive_datetime()) :: String.t()
```

Converts the given naive datetime to a string according to its calendar.

#### Examples

    iex> NaiveDateTime.to_string(~N[2000-02-28 23:00:13])
    "2000-02-28 23:00:13"
    iex> NaiveDateTime.to_string(~N[2000-02-28 23:00:13.001])
    "2000-02-28 23:00:13.001"
    iex> NaiveDateTime.to_string(~N[-0100-12-15 03:20:31])
    "-0100-12-15 03:20:31"

This function can also be used to convert a DateTime to a string without
the time zone information:

    iex> dt = %DateTime{year: 2000, month: 2, day: 29, zone_abbr: "CET",
    ...>                hour: 23, minute: 0, second: 7, microsecond: {0, 0},
    ...>                utc_offset: 3600, std_offset: 0, time_zone: "Europe/Warsaw"}
    iex> NaiveDateTime.to_string(dt)
    "2000-02-29 23:00:07"


### to_time(map)

```elixir
@spec to_time(Calendar.naive_datetime()) :: Time.t()
```

Converts a `NaiveDateTime` into `Time`.

Because `Time` does not hold date information,
data will be lost during the conversion.

#### Examples

    iex> NaiveDateTime.to_time(~N[2002-01-13 23:00:07])
    ~T[23:00:07]


### truncate(naive_datetime, precision)
*(since 1.6.0)* 
```elixir
@spec truncate(t(), :microsecond | :millisecond | :second) :: t()
```

Returns the given naive datetime with the microsecond field truncated to the
given precision (`:microsecond`, `:millisecond` or `:second`).

The given naive datetime is returned unchanged if it already has lower precision
than the given precision.

#### Examples

    iex> NaiveDateTime.truncate(~N[2017-11-06 00:23:51.123456], :microsecond)
    ~N[2017-11-06 00:23:51.123456]
    
    iex> NaiveDateTime.truncate(~N[2017-11-06 00:23:51.123456], :millisecond)
    ~N[2017-11-06 00:23:51.123]
    
    iex> NaiveDateTime.truncate(~N[2017-11-06 00:23:51.123456], :second)
    ~N[2017-11-06 00:23:51]


### utc_now(calendar_or_time_unit \\ Calendar.ISO)
*(since 1.4.0)* 
```elixir
@spec utc_now(Calendar.calendar() | :native | :microsecond | :millisecond | :second) ::
  t()
```

Returns the current naive datetime in UTC.

Prefer using `DateTime.utc_now/0` when possible as, opposite
to `NaiveDateTime`, it will keep the time zone information.

You can also provide a time unit to automatically truncate
the naive datetime. This is available since v1.15.0.

#### Examples

    iex> naive_datetime = NaiveDateTime.utc_now()
    iex> naive_datetime.year >= 2016
    true
    
    iex> naive_datetime = NaiveDateTime.utc_now(:second)
    iex> naive_datetime.microsecond
    {0, 0}


### utc_now(time_unit, calendar)
*(since 1.15.0)* 
```elixir
@spec utc_now(:native | :microsecond | :millisecond | :second, Calendar.calendar()) ::
  t()
```

Returns the current naive datetime in UTC, supporting a specific
calendar and precision.

Prefer using `DateTime.utc_now/2` when possible as, opposite
to `NaiveDateTime`, it will keep the time zone information.

#### Examples

    iex> naive_datetime = NaiveDateTime.utc_now(:second, Calendar.ISO)
    iex> naive_datetime.year >= 2016
    true
    
    iex> naive_datetime = NaiveDateTime.utc_now(:second, Calendar.ISO)
    iex> naive_datetime.microsecond
    {0, 0}




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
