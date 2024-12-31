# Date 
(Elixir v1.18.0-dev)

A Date struct and functions.

The Date struct contains the fields year, month, day and calendar.
New dates can be built with the `new/3` function or using the
`~D` (see `sigil_D/2`) sigil:

    iex> ~D[2000-01-01]
    ~D[2000-01-01]

Both `new/3` and sigil return a struct where the date fields can
be accessed directly:

    iex> date = ~D[2000-01-01]
    iex> date.year
    2000
    iex> date.month
    1

The functions on this module work with the `Date` struct as well
as any struct that contains the same fields as the `Date` struct,
such as `NaiveDateTime` and `DateTime`. Such functions expect
`t:Calendar.date/0` in their typespecs (instead of `t:t/0`).

Developers should avoid creating the Date structs directly
and instead rely on the functions provided by this module as well
as the ones in third-party calendar libraries.

## Comparing dates

Comparisons in Elixir using `==/2`, `>/2`, `</2` and similar are structural
and based on the `Date` struct fields. For proper comparison between
dates, use the `compare/2` function. The existence of the `compare/2`
function in this module also allows using `Enum.min/2` and `Enum.max/2`
functions to get the minimum and maximum date of an `Enum`. For example:

    iex>  Enum.min([~D[2017-03-31], ~D[2017-04-01]], Date)
    ~D[2017-03-31]

## Using epochs

The `add/2`, `diff/2` and `shift/2` functions can be used for computing dates
or retrieving the number of days between instants. For example, if there
is an interest in computing the number of days from the Unix epoch
(1970-01-01):

    iex> Date.diff(~D[2010-04-17], ~D[1970-01-01])
    14716
    
    iex> Date.add(~D[1970-01-01], 14716)
    ~D[2010-04-17]
    
    iex> Date.shift(~D[1970-01-01], year: 40, month: 3, week: 2, day: 2)
    ~D[2010-04-17]

Those functions are optimized to deal with common epochs, such
as the Unix Epoch above or the Gregorian Epoch (0000-01-01).

## Types

### t()

```elixir
@type t() :: %Date{
  calendar: Calendar.calendar(),
  day: Calendar.day(),
  month: Calendar.month(),
  year: Calendar.year()
}
```



## Functions

### add(date, days)
*(since 1.5.0)* 
```elixir
@spec add(Calendar.date(), integer()) :: t()
```

Adds the number of days to the given `date`.

The days are counted as Gregorian days. The date is returned in the same
calendar as it was given in.

To shift a date by a `Duration` and according to its underlying calendar, use `Date.shift/2`.

#### Examples

    iex> Date.add(~D[2000-01-03], -2)
    ~D[2000-01-01]
    iex> Date.add(~D[2000-01-01], 2)
    ~D[2000-01-03]
    iex> Date.add(~N[2000-01-01 09:00:00], 2)
    ~D[2000-01-03]
    iex> Date.add(~D[-0010-01-01], -2)
    ~D[-0011-12-30]

### after?(date1, date2)
*(since 1.15.0)* 
```elixir
@spec after?(Calendar.date(), Calendar.date()) :: boolean()
```

Returns `true` if the first date is strictly later than the second.

#### Examples

    iex> Date.after?(~D[2022-02-02], ~D[2021-01-01])
    true
    iex> Date.after?(~D[2021-01-01], ~D[2021-01-01])
    false
    iex> Date.after?(~D[2021-01-01], ~D[2022-02-02])
    false

### before?(date1, date2)
*(since 1.15.0)* 
```elixir
@spec before?(Calendar.date(), Calendar.date()) :: boolean()
```

Returns `true` if the first date is strictly earlier than the second.

#### Examples

    iex> Date.before?(~D[2021-01-01], ~D[2022-02-02])
    true
    iex> Date.before?(~D[2021-01-01], ~D[2021-01-01])
    false
    iex> Date.before?(~D[2022-02-02], ~D[2021-01-01])
    false

### beginning_of_month(date)
*(since 1.11.0)* 
```elixir
@spec beginning_of_month(Calendar.date()) :: t()
```

Calculates a date that is the first day of the month for the given `date`.

#### Examples

    iex> Date.beginning_of_month(~D[2000-01-31])
    ~D[2000-01-01]
    iex> Date.beginning_of_month(~D[2000-01-01])
    ~D[2000-01-01]
    iex> Date.beginning_of_month(~N[2000-01-31 01:23:45])
    ~D[2000-01-01]

### beginning_of_week(date, starting_on \\ :default)
*(since 1.11.0)* 
```elixir
@spec beginning_of_week(Calendar.date(), starting_on :: :default | atom()) :: t()
```

Calculates a date that is the first day of the week for the given `date`.

If the day is already the first day of the week, it returns the
day itself. For the built-in ISO calendar, the week starts on Monday.
A weekday rather than `:default` can be given as `starting_on`.

#### Examples

    iex> Date.beginning_of_week(~D[2020-07-11])
    ~D[2020-07-06]
    iex> Date.beginning_of_week(~D[2020-07-06])
    ~D[2020-07-06]
    iex> Date.beginning_of_week(~D[2020-07-11], :sunday)
    ~D[2020-07-05]
    iex> Date.beginning_of_week(~D[2020-07-11], :saturday)
    ~D[2020-07-11]
    iex> Date.beginning_of_week(~N[2020-07-11 01:23:45])
    ~D[2020-07-06]

### compare(date1, date2)
*(since 1.4.0)* 
```elixir
@spec compare(Calendar.date(), Calendar.date()) :: :lt | :eq | :gt
```

Compares two date structs.

Returns `:gt` if first date is later than the second
and `:lt` for vice versa. If the two dates are equal
`:eq` is returned.

#### Examples

    iex> Date.compare(~D[2016-04-16], ~D[2016-04-28])
    :lt

This function can also be used to compare across more
complex calendar types by considering only the date fields:

    iex> Date.compare(~D[2016-04-16], ~N[2016-04-28 01:23:45])
    :lt
    iex> Date.compare(~D[2016-04-16], ~N[2016-04-16 01:23:45])
    :eq
    iex> Date.compare(~N[2016-04-16 12:34:56], ~N[2016-04-16 01:23:45])
    :eq

### convert(date, calendar)
*(since 1.5.0)* 
```elixir
@spec convert(Calendar.date(), Calendar.calendar()) ::
  {:ok, t()} | {:error, :incompatible_calendars}
```

Converts the given `date` from its calendar to the given `calendar`.

Returns `{:ok, date}` if the calendars are compatible,
or `{:error, :incompatible_calendars}` if they are not.

See also `Calendar.compatible_calendars?/2`.

#### Examples

Imagine someone implements `Calendar.Holocene`, a calendar based on the
Gregorian calendar that adds exactly 10,000 years to the current Gregorian
year:

    iex> Date.convert(~D[2000-01-01], Calendar.Holocene)
    {:ok, %Date{calendar: Calendar.Holocene, year: 12000, month: 1, day: 1}}

### convert!(date, calendar)
*(since 1.5.0)* 
```elixir
@spec convert!(Calendar.date(), Calendar.calendar()) :: t()
```

Similar to `Date.convert/2`, but raises an `ArgumentError`
if the conversion between the two calendars is not possible.

#### Examples

Imagine someone implements `Calendar.Holocene`, a calendar based on the
Gregorian calendar that adds exactly 10,000 years to the current Gregorian
year:

    iex> Date.convert!(~D[2000-01-01], Calendar.Holocene)
    %Date{calendar: Calendar.Holocene, year: 12000, month: 1, day: 1}

### day_of_era(date)
*(since 1.8.0)* 
```elixir
@spec day_of_era(Calendar.date()) :: {Calendar.day(), non_neg_integer()}
```

Calculates the day-of-era and era for a given
calendar `date`.

Returns a tuple `{day, era}` representing the
day within the era and the era number.

#### Examples

    iex> Date.day_of_era(~D[0001-01-01])
    {1, 1}
    
    iex> Date.day_of_era(~D[0000-12-31])
    {1, 0}

### day_of_week(date, starting_on \\ :default)
*(since 1.4.0)* 
```elixir
@spec day_of_week(Calendar.date(), starting_on :: :default | atom()) ::
  Calendar.day_of_week()
```

Calculates the day of the week of a given `date`.

Returns the day of the week as an integer. For the ISO 8601
calendar (the default), it is an integer from 1 to 7, where
1 is Monday and 7 is Sunday.

An optional `starting_on` value may be supplied, which
configures the weekday the week starts on. The default value
for it is `:default`, which translates to `:monday` for the
built-in ISO calendar. Any other weekday may be given to.

#### Examples

    iex> Date.day_of_week(~D[2016-10-31])
    1
    iex> Date.day_of_week(~D[2016-11-01])
    2
    iex> Date.day_of_week(~N[2016-11-01 01:23:45])
    2
    iex> Date.day_of_week(~D[-0015-10-30])
    3
    
    iex> Date.day_of_week(~D[2016-10-31], :sunday)
    2
    iex> Date.day_of_week(~D[2016-11-01], :sunday)
    3
    iex> Date.day_of_week(~N[2016-11-01 01:23:45], :sunday)
    3
    iex> Date.day_of_week(~D[-0015-10-30], :sunday)
    4

### day_of_year(date)
*(since 1.8.0)* 
```elixir
@spec day_of_year(Calendar.date()) :: Calendar.day()
```

Calculates the day of the year of a given `date`.

Returns the day of the year as an integer. For the ISO 8601
calendar (the default), it is an integer from 1 to 366.

#### Examples

    iex> Date.day_of_year(~D[2016-01-01])
    1
    iex> Date.day_of_year(~D[2016-11-01])
    306
    iex> Date.day_of_year(~D[-0015-10-30])
    303
    iex> Date.day_of_year(~D[2004-12-31])
    366

### days_in_month(date)
*(since 1.4.0)* 
```elixir
@spec days_in_month(Calendar.date()) :: Calendar.day()
```

Returns the number of days in the given `date` month.

#### Examples

    iex> Date.days_in_month(~D[1900-01-13])
    31
    iex> Date.days_in_month(~D[1900-02-09])
    28
    iex> Date.days_in_month(~N[2000-02-20 01:23:45])
    29

### diff(date1, date2)
*(since 1.5.0)* 
```elixir
@spec diff(Calendar.date(), Calendar.date()) :: integer()
```

Calculates the difference between two dates, in a full number of days.

It returns the number of Gregorian days between the dates. Only `Date`
structs that follow the same or compatible calendars can be compared
this way. If two calendars are not compatible, it will raise.

#### Examples

    iex> Date.diff(~D[2000-01-03], ~D[2000-01-01])
    2
    iex> Date.diff(~D[2000-01-01], ~D[2000-01-03])
    -2
    iex> Date.diff(~D[0000-01-02], ~D[-0001-12-30])
    3
    iex> Date.diff(~D[2000-01-01], ~N[2000-01-03 09:00:00])
    -2

### end_of_month(date)
*(since 1.11.0)* 
```elixir
@spec end_of_month(Calendar.date()) :: t()
```

Calculates a date that is the last day of the month for the given `date`.

#### Examples

    iex> Date.end_of_month(~D[2000-01-01])
    ~D[2000-01-31]
    iex> Date.end_of_month(~D[2000-01-31])
    ~D[2000-01-31]
    iex> Date.end_of_month(~N[2000-01-01 01:23:45])
    ~D[2000-01-31]

### end_of_week(date, starting_on \\ :default)
*(since 1.11.0)* 
```elixir
@spec end_of_week(Calendar.date(), starting_on :: :default | atom()) :: t()
```

Calculates a date that is the last day of the week for the given `date`.

If the day is already the last day of the week, it returns the
day itself. For the built-in ISO calendar, the week ends on Sunday.
A weekday rather than `:default` can be given as `starting_on`.

#### Examples

    iex> Date.end_of_week(~D[2020-07-11])
    ~D[2020-07-12]
    iex> Date.end_of_week(~D[2020-07-05])
    ~D[2020-07-05]
    iex> Date.end_of_week(~D[2020-07-06], :sunday)
    ~D[2020-07-11]
    iex> Date.end_of_week(~D[2020-07-06], :saturday)
    ~D[2020-07-10]
    iex> Date.end_of_week(~N[2020-07-11 01:23:45])
    ~D[2020-07-12]

### from_erl(tuple, calendar \\ Calendar.ISO)

```elixir
@spec from_erl(:calendar.date(), Calendar.calendar()) :: {:ok, t()} | {:error, atom()}
```

Converts an Erlang date tuple to a `Date` struct.

Only supports converting dates which are in the ISO calendar,
or other calendars in which the days also start at midnight.
Attempting to convert dates from other calendars will return an error tuple.

#### Examples

    iex> Date.from_erl({2000, 1, 1})
    {:ok, ~D[2000-01-01]}
    iex> Date.from_erl({2000, 13, 1})
    {:error, :invalid_date}

### from_erl!(tuple, calendar \\ Calendar.ISO)

```elixir
@spec from_erl!(:calendar.date(), Calendar.calendar()) :: t()
```

Converts an Erlang date tuple but raises for invalid dates.

#### Examples

    iex> Date.from_erl!({2000, 1, 1})
    ~D[2000-01-01]
    iex> Date.from_erl!({2000, 13, 1})
    ** (ArgumentError) cannot convert {2000, 13, 1} to date, reason: :invalid_date

### from_gregorian_days(days, calendar \\ Calendar.ISO)
*(since 1.11.0)* 
```elixir
@spec from_gregorian_days(integer(), Calendar.calendar()) :: t()
```

Converts a number of gregorian days to a `Date` struct.

#### Examples

    iex> Date.from_gregorian_days(1)
    ~D[0000-01-02]
    iex> Date.from_gregorian_days(730_485)
    ~D[2000-01-01]
    iex> Date.from_gregorian_days(-1)
    ~D[-0001-12-31]

### from_iso8601(string, calendar \\ Calendar.ISO)

```elixir
@spec from_iso8601(String.t(), Calendar.calendar()) :: {:ok, t()} | {:error, atom()}
```

Parses the extended "Dates" format described by
[ISO 8601:2019](https://en.wikipedia.org/wiki/ISO_8601).

The year parsed by this function is limited to four digits.

#### Examples

    iex> Date.from_iso8601("2015-01-23")
    {:ok, ~D[2015-01-23]}
    
    iex> Date.from_iso8601("2015:01:23")
    {:error, :invalid_format}
    
    iex> Date.from_iso8601("2015-01-32")
    {:error, :invalid_date}

### from_iso8601!(string, calendar \\ Calendar.ISO)

```elixir
@spec from_iso8601!(String.t(), Calendar.calendar()) :: t()
```

Parses the extended "Dates" format described by
[ISO 8601:2019](https://en.wikipedia.org/wiki/ISO_8601).

Raises if the format is invalid.

#### Examples

    iex> Date.from_iso8601!("2015-01-23")
    ~D[2015-01-23]
    iex> Date.from_iso8601!("2015:01:23")
    ** (ArgumentError) cannot parse "2015:01:23" as date, reason: :invalid_format

### leap_year?(date)
*(since 1.4.0)* 
```elixir
@spec leap_year?(Calendar.date()) :: boolean()
```

Returns `true` if the year in the given `date` is a leap year.

#### Examples

    iex> Date.leap_year?(~D[2000-01-01])
    true
    iex> Date.leap_year?(~D[2001-01-01])
    false
    iex> Date.leap_year?(~D[2004-01-01])
    true
    iex> Date.leap_year?(~D[1900-01-01])
    false
    iex> Date.leap_year?(~N[2004-01-01 01:23:45])
    true

### months_in_year(date)
*(since 1.7.0)* 
```elixir
@spec months_in_year(Calendar.date()) :: Calendar.month()
```

Returns the number of months in the given `date` year.

#### Example

    iex> Date.months_in_year(~D[1900-01-13])
    12

### new(year, month, day, calendar \\ Calendar.ISO)

```elixir
@spec new(Calendar.year(), Calendar.month(), Calendar.day(), Calendar.calendar()) ::
  {:ok, t()} | {:error, atom()}
```

Builds a new ISO date.

Expects all values to be integers. Returns `{:ok, date}` if each
entry fits its appropriate range, returns `{:error, reason}` otherwise.

#### Examples

    iex> Date.new(2000, 1, 1)
    {:ok, ~D[2000-01-01]}
    iex> Date.new(2000, 13, 1)
    {:error, :invalid_date}
    iex> Date.new(2000, 2, 29)
    {:ok, ~D[2000-02-29]}
    
    iex> Date.new(2000, 2, 30)
    {:error, :invalid_date}
    iex> Date.new(2001, 2, 29)
    {:error, :invalid_date}

### new!(year, month, day, calendar \\ Calendar.ISO)
*(since 1.11.0)* 
```elixir
@spec new!(Calendar.year(), Calendar.month(), Calendar.day(), Calendar.calendar()) ::
  t()
```

Builds a new ISO date.

Expects all values to be integers. Returns `date` if each
entry fits its appropriate range, raises if the date is invalid.

#### Examples

    iex> Date.new!(2000, 1, 1)
    ~D[2000-01-01]
    iex> Date.new!(2000, 13, 1)
    ** (ArgumentError) cannot build date, reason: :invalid_date
    iex> Date.new!(2000, 2, 29)
    ~D[2000-02-29]

### quarter_of_year(date)
*(since 1.8.0)* 
```elixir
@spec quarter_of_year(Calendar.date()) :: non_neg_integer()
```

Calculates the quarter of the year of a given `date`.

Returns the day of the year as an integer. For the ISO 8601
calendar (the default), it is an integer from 1 to 4.

#### Examples

    iex> Date.quarter_of_year(~D[2016-10-31])
    4
    iex> Date.quarter_of_year(~D[2016-01-01])
    1
    iex> Date.quarter_of_year(~N[2016-04-01 01:23:45])
    2
    iex> Date.quarter_of_year(~D[-0015-09-30])
    3

### range(first, last)
*(since 1.5.0)* 
```elixir
@spec range(Calendar.date(), Calendar.date()) :: Date.Range.t()
```

Returns a range of dates.

A range of dates represents a discrete number of dates where
the first and last values are dates with matching calendars.

Ranges of dates can be increasing (`first <= last`) and are
always inclusive. For a decreasing range, use `range/3` with
a step of -1 as first argument.

#### Examples

    iex> Date.range(~D[1999-01-01], ~D[2000-01-01])
    Date.range(~D[1999-01-01], ~D[2000-01-01])

A range of dates implements the `Enumerable` protocol, which means
functions in the `Enum` module can be used to work with
ranges:

    iex> range = Date.range(~D[2001-01-01], ~D[2002-01-01])
    iex> range
    Date.range(~D[2001-01-01], ~D[2002-01-01])
    iex> Enum.count(range)
    366
    iex> ~D[2001-02-01] in range
    true
    iex> Enum.take(range, 3)
    [~D[2001-01-01], ~D[2001-01-02], ~D[2001-01-03]]

### range(first, last, step)
*(since 1.12.0)* 
```elixir
@spec range(Calendar.date(), Calendar.date(), step :: pos_integer() | neg_integer()) ::
  Date.Range.t()
```

Returns a range of dates with a step.

#### Examples

    iex> range = Date.range(~D[2001-01-01], ~D[2002-01-01], 2)
    iex> range
    Date.range(~D[2001-01-01], ~D[2002-01-01], 2)
    iex> Enum.count(range)
    183
    iex> ~D[2001-01-03] in range
    true
    iex> Enum.take(range, 3)
    [~D[2001-01-01], ~D[2001-01-03], ~D[2001-01-05]]

### shift(date, duration)
*(since 1.17.0)* 
```elixir
@spec shift(Calendar.date(), Duration.t() | [unit_pair]) :: t()
when unit_pair:
       {:year, integer()}
       | {:month, integer()}
       | {:week, integer()}
       | {:day, integer()}
```

Shifts given `date` by `duration` according to its calendar.

Allowed units are: `:year`, `:month`, `:week`, `:day`.

When using the default ISO calendar, durations are collapsed and
applied in the order of months and then days:

- when shifting by 1 year and 2 months the date is actually shifted by 14 months
- when shifting by 2 weeks and 3 days the date is shifted by 17 days

When shifting by month, days are rounded down to the nearest valid date.

Raises an `ArgumentError` when called with time scale units.

#### Examples

    iex> Date.shift(~D[2016-01-03], month: 2)
    ~D[2016-03-03]
    iex> Date.shift(~D[2016-01-30], month: -1)
    ~D[2015-12-30]
    iex> Date.shift(~D[2016-01-31], year: 4, day: 1)
    ~D[2020-02-01]
    iex> Date.shift(~D[2016-01-03], Duration.new!(month: 2))
    ~D[2016-03-03]
    
    # leap years
    iex> Date.shift(~D[2024-02-29], year: 1)
    ~D[2025-02-28]
    iex> Date.shift(~D[2024-02-29], year: 4)
    ~D[2028-02-29]
    
    # rounding down
    iex> Date.shift(~D[2015-01-31], month: 1)
    ~D[2015-02-28]

### to_erl(date)

```elixir
@spec to_erl(Calendar.date()) :: :calendar.date()
```

Converts the given `date` to an Erlang date tuple.

Only supports converting dates which are in the ISO calendar,
or other calendars in which the days also start at midnight.
Attempting to convert dates from other calendars will raise.

#### Examples

    iex> Date.to_erl(~D[2000-01-01])
    {2000, 1, 1}
    
    iex> Date.to_erl(~N[2000-01-01 00:00:00])
    {2000, 1, 1}

### to_gregorian_days(date)
*(since 1.11.0)* 
```elixir
@spec to_gregorian_days(Calendar.date()) :: integer()
```

Converts a `date` struct to a number of gregorian days.

#### Examples

    iex> Date.to_gregorian_days(~D[0000-01-02])
    1
    iex> Date.to_gregorian_days(~D[2000-01-01])
    730_485
    iex> Date.to_gregorian_days(~N[2000-01-01 00:00:00])
    730_485

### to_iso8601(date, format \\ :extended)

```elixir
@spec to_iso8601(Calendar.date(), :extended | :basic) :: String.t()
```

Converts the given `date` to
[ISO 8601:2019](https://en.wikipedia.org/wiki/ISO_8601).

By default, `Date.to_iso8601/2` returns dates formatted in the "extended"
format, for human readability. It also supports the "basic" format through passing the `:basic` option.

Only supports converting dates which are in the ISO calendar,
or other calendars in which the days also start at midnight.
Attempting to convert dates from other calendars will raise an `ArgumentError`.

#### Examples

    iex> Date.to_iso8601(~D[2000-02-28])
    "2000-02-28"
    
    iex> Date.to_iso8601(~D[2000-02-28], :basic)
    "20000228"
    
    iex> Date.to_iso8601(~N[2000-02-28 00:00:00])
    "2000-02-28"

### to_string(date)

```elixir
@spec to_string(Calendar.date()) :: String.t()
```

Converts the given date to a string according to its calendar.

#### Examples

    iex> Date.to_string(~D[2000-02-28])
    "2000-02-28"
    iex> Date.to_string(~N[2000-02-28 01:23:45])
    "2000-02-28"
    iex> Date.to_string(~D[-0100-12-15])
    "-0100-12-15"

### utc_today(calendar \\ Calendar.ISO)
*(since 1.4.0)* 
```elixir
@spec utc_today(Calendar.calendar()) :: t()
```

Returns the current date in UTC.

#### Examples

    iex> date = Date.utc_today()
    iex> date.year >= 2016
    true

### year_of_era(date)
*(since 1.8.0)* 
```elixir
@spec year_of_era(Calendar.date()) :: {Calendar.year(), non_neg_integer()}
```

Calculates the year-of-era and era for a given
calendar year.

Returns a tuple `{year, era}` representing the
year within the era and the era number.

#### Examples

    iex> Date.year_of_era(~D[0001-01-01])
    {1, 1}
    iex> Date.year_of_era(~D[0000-12-31])
    {1, 0}
    iex> Date.year_of_era(~D[-0001-01-01])
    {2, 0}



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
