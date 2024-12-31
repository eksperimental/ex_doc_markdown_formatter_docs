# Calendar.ISO 
(Elixir v1.18.0-dev)

The default calendar implementation, a Gregorian calendar following ISO 8601.

This calendar implements a proleptic Gregorian calendar and
is therefore compatible with the calendar used in most countries
today. The proleptic means the Gregorian rules for leap years are
applied for all time, consequently the dates give different results
before the year 1583 from when the Gregorian calendar was adopted.

## ISO 8601 compliance

The ISO 8601 specification is feature-rich, but allows applications
to selectively implement most parts of it. The choices Elixir makes
are catalogued below.

### Features

The standard library supports a minimal set of possible ISO 8601 features.
Specifically, the parser only supports calendar dates and does not support
ordinal and week formats. Additionally, it supports parsing ISO 8601
formatted durations, including negative time units and fractional seconds.

By default Elixir only parses extended-formatted date/times. You can opt-in
to parse basic-formatted date/times.

`NaiveDateTime.to_iso8601/2` and `DateTime.to_iso8601/2` allow you to produce
either basic or extended formatted strings, and `Calendar.strftime/2` allows
you to format datetimes however else you desire.

Elixir does not support reduced accuracy formats (for example, a date without
the day component) nor decimal precisions in the lowest component (such as
`10:01:25,5`).

#### Examples

Elixir expects the extended format by default when parsing:

    iex> Calendar.ISO.parse_naive_datetime("2015-01-23T23:50:07")
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_naive_datetime("20150123T235007")
    {:error, :invalid_format}

Parsing can be restricted to basic if desired:

    iex> Calendar.ISO.parse_naive_datetime("20150123T235007Z", :basic)
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_naive_datetime("20150123T235007Z", :extended)
    {:error, :invalid_format}

Only calendar dates are supported in parsing; ordinal and week dates are not.

    iex> Calendar.ISO.parse_date("2015-04-15")
    {:ok, {2015, 4, 15}}
    iex> Calendar.ISO.parse_date("2015-105")
    {:error, :invalid_format}
    iex> Calendar.ISO.parse_date("2015-W16")
    {:error, :invalid_format}
    iex> Calendar.ISO.parse_date("2015-W016-3")
    {:error, :invalid_format}

Years, months, days, hours, minutes, and seconds must be fully specified:

    iex> Calendar.ISO.parse_date("2015-04-15")
    {:ok, {2015, 4, 15}}
    iex> Calendar.ISO.parse_date("2015-04")
    {:error, :invalid_format}
    iex> Calendar.ISO.parse_date("2015")
    {:error, :invalid_format}
    
    iex> Calendar.ISO.parse_time("23:50:07.0123456")
    {:ok, {23, 50, 7, {12345, 6}}}
    iex> Calendar.ISO.parse_time("23:50:07")
    {:ok, {23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_time("23:50")
    {:error, :invalid_format}
    iex> Calendar.ISO.parse_time("23")
    {:error, :invalid_format}

### Extensions

The parser and formatter adopt one ISO 8601 extension: extended year notation.

This allows dates to be prefixed with a `+` or `-` sign, extending the range of
expressible years from the default (`0000..9999`) to `-9999..9999`. Elixir still
restricts years in this format to four digits.

#### Examples

    iex> Calendar.ISO.parse_date("-2015-01-23")
    {:ok, {-2015, 1, 23}}
    iex> Calendar.ISO.parse_date("+2015-01-23")
    {:ok, {2015, 1, 23}}
    
    iex> Calendar.ISO.parse_naive_datetime("-2015-01-23 23:50:07")
    {:ok, {-2015, 1, 23, 23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_naive_datetime("+2015-01-23 23:50:07")
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}}
    
    iex> Calendar.ISO.parse_utc_datetime("-2015-01-23 23:50:07Z")
    {:ok, {-2015, 1, 23, 23, 50, 7, {0, 0}}, 0}
    iex> Calendar.ISO.parse_utc_datetime("+2015-01-23 23:50:07Z")
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}, 0}

### Additions

ISO 8601 does not allow a whitespace instead of `T` as a separator
between date and times, both when parsing and formatting.
This is a common enough representation, Elixir allows it during parsing.

The formatting of dates in `NaiveDateTime.to_iso8601/1` and `DateTime.to_iso8601/1`
do produce specification-compliant string representations using the `T` separator.

#### Examples

    iex> Calendar.ISO.parse_naive_datetime("2015-01-23 23:50:07.0123456")
    {:ok, {2015, 1, 23, 23, 50, 7, {12345, 6}}}
    iex> Calendar.ISO.parse_naive_datetime("2015-01-23T23:50:07.0123456")
    {:ok, {2015, 1, 23, 23, 50, 7, {12345, 6}}}
    
    iex> Calendar.ISO.parse_utc_datetime("2015-01-23 23:50:07.0123456Z")
    {:ok, {2015, 1, 23, 23, 50, 7, {12345, 6}}, 0}
    iex> Calendar.ISO.parse_utc_datetime("2015-01-23T23:50:07.0123456Z")
    {:ok, {2015, 1, 23, 23, 50, 7, {12345, 6}}, 0}


## Types

### bce()

```elixir
@type bce() :: 0
```

"Before the Current Era" or "Before the Common Era" (BCE), for those years less than `1`.


### ce()

```elixir
@type ce() :: 1
```

The "Current Era" or the "Common Era" (CE) which starts in year `1`.


### day()

```elixir
@type day() :: 1..31
```



### day_of_week()

```elixir
@type day_of_week() :: 1..7
```

Integer that represents the day of the week, where 1 is Monday and 7 is Sunday.


### day_of_year()

```elixir
@type day_of_year() :: 1..366
```



### era()

```elixir
@type era() :: bce() | ce()
```

The calendar era.

The ISO calendar has two eras:

- [CE](\`t:ce/0\`) - which starts in year `1` and is defined as era `1`.
- [BCE](\`t:bce/0\`) - for those years less than `1` and is defined as era `0`.


### format()

```elixir
@type format() :: :basic | :extended
```



### hour()

```elixir
@type hour() :: 0..23
```



### microsecond()

```elixir
@type microsecond() :: {0..999_999, 0..6}
```

Microseconds with stored precision.

The precision represents the number of digits that must be used when
representing the microseconds to external format. If the precision is 0,
it means microseconds must be skipped.


### minute()

```elixir
@type minute() :: 0..59
```



### month()

```elixir
@type month() :: 1..12
```



### quarter_of_year()

```elixir
@type quarter_of_year() :: 1..4
```



### second()

```elixir
@type second() :: 0..59
```



### utc_offset()

```elixir
@type utc_offset() :: integer()
```



### weekday()

```elixir
@type weekday() ::
  :monday | :tuesday | :wednesday | :thursday | :friday | :saturday | :sunday
```



### year()

```elixir
@type year() :: -9999..9999
```



### year_of_era()

```elixir
@type year_of_era() :: {1..10000, era()}
```



## Functions

### date_to_string(year, month, day, format \\ :extended)
*(since 1.4.0)* 
```elixir
@spec date_to_string(year(), month(), day(), :basic | :extended) :: String.t()
```

Converts the given date into a string.

By default, returns dates formatted in the "extended" format,
for human readability. It also supports the "basic" format
by passing the `:basic` option.

#### Examples

    iex> Calendar.ISO.date_to_string(2015, 2, 28)
    "2015-02-28"
    iex> Calendar.ISO.date_to_string(2017, 8, 1)
    "2017-08-01"
    iex> Calendar.ISO.date_to_string(-99, 1, 31)
    "-0099-01-31"
    
    iex> Calendar.ISO.date_to_string(2015, 2, 28, :basic)
    "20150228"
    iex> Calendar.ISO.date_to_string(-99, 1, 31, :basic)
    "-00990131"


### datetime_to_string(year, month, day, hour, minute, second, microsecond, time_zone, zone_abbr, utc_offset, std_offset, format \\ :extended)
*(since 1.4.0)* 
```elixir
@spec datetime_to_string(
  year(),
  month(),
  day(),
  Calendar.hour(),
  Calendar.minute(),
  Calendar.second(),
  Calendar.microsecond(),
  Calendar.time_zone(),
  Calendar.zone_abbr(),
  Calendar.utc_offset(),
  Calendar.std_offset(),
  :basic | :extended
) :: String.t()
```

Converts the datetime (with time zone) into a string.

By default, returns datetimes formatted in the "extended" format,
for human readability. It also supports the "basic" format
by passing the `:basic` option.

#### Examples

    iex> time_zone = "Etc/UTC"
    iex> Calendar.ISO.datetime_to_string(2017, 8, 1, 1, 2, 3, {4, 5}, time_zone, "UTC", 0, 0)
    "2017-08-01 01:02:03.00000Z"
    iex> Calendar.ISO.datetime_to_string(2017, 8, 1, 1, 2, 3, {4, 5}, time_zone, "UTC", 3600, 0)
    "2017-08-01 01:02:03.00000+01:00"
    iex> Calendar.ISO.datetime_to_string(2017, 8, 1, 1, 2, 3, {4, 5}, time_zone, "UTC", 3600, 3600)
    "2017-08-01 01:02:03.00000+02:00"
    
    iex> time_zone = "Europe/Berlin"
    iex> Calendar.ISO.datetime_to_string(2017, 8, 1, 1, 2, 3, {4, 5}, time_zone, "CET", 3600, 0)
    "2017-08-01 01:02:03.00000+01:00 CET Europe/Berlin"
    iex> Calendar.ISO.datetime_to_string(2017, 8, 1, 1, 2, 3, {4, 5}, time_zone, "CDT", 3600, 3600)
    "2017-08-01 01:02:03.00000+02:00 CDT Europe/Berlin"
    
    iex> time_zone = "America/Los_Angeles"
    iex> Calendar.ISO.datetime_to_string(2015, 2, 28, 1, 2, 3, {4, 5}, time_zone, "PST", -28800, 0)
    "2015-02-28 01:02:03.00000-08:00 PST America/Los_Angeles"
    iex> Calendar.ISO.datetime_to_string(2015, 2, 28, 1, 2, 3, {4, 5}, time_zone, "PDT", -28800, 3600)
    "2015-02-28 01:02:03.00000-07:00 PDT America/Los_Angeles"
    
    iex> time_zone = "Europe/Berlin"
    iex> Calendar.ISO.datetime_to_string(2017, 8, 1, 1, 2, 3, {4, 5}, time_zone, "CET", 3600, 0, :basic)
    "20170801 010203.00000+0100 CET Europe/Berlin"


### day_of_era(year, month, day)
*(since 1.8.0)* 
```elixir
@spec day_of_era(year(), month(), day()) :: Calendar.day_of_era()
```

Calculates the day and era from the given `year`, `month`, and `day`.

#### Examples

    iex> Calendar.ISO.day_of_era(0, 1, 1)
    {366, 0}
    iex> Calendar.ISO.day_of_era(1, 1, 1)
    {1, 1}
    iex> Calendar.ISO.day_of_era(0, 12, 31)
    {1, 0}
    iex> Calendar.ISO.day_of_era(0, 12, 30)
    {2, 0}
    iex> Calendar.ISO.day_of_era(-1, 12, 31)
    {367, 0}


### day_of_week(year, month, day, starting_on)
*(since 1.11.0)* 
```elixir
@spec day_of_week(year(), month(), day(), :default | weekday()) ::
  {day_of_week(), 1, 7}
```

Calculates the day of the week from the given `year`, `month`, and `day`.

It is an integer from 1 to 7, where 1 is the given `starting_on` weekday.
For example, if `starting_on` is set to `:monday`, then 1 is Monday and
7 is Sunday.

`starting_on` can also be `:default`, which is equivalent to `:monday`.

#### Examples

    iex> Calendar.ISO.day_of_week(2016, 10, 31, :monday)
    {1, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 1, :monday)
    {2, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 2, :monday)
    {3, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 3, :monday)
    {4, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 4, :monday)
    {5, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 5, :monday)
    {6, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 6, :monday)
    {7, 1, 7}
    iex> Calendar.ISO.day_of_week(-99, 1, 31, :monday)
    {4, 1, 7}
    
    iex> Calendar.ISO.day_of_week(2016, 10, 31, :sunday)
    {2, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 1, :sunday)
    {3, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 2, :sunday)
    {4, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 3, :sunday)
    {5, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 4, :sunday)
    {6, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 5, :sunday)
    {7, 1, 7}
    iex> Calendar.ISO.day_of_week(2016, 11, 6, :sunday)
    {1, 1, 7}
    iex> Calendar.ISO.day_of_week(-99, 1, 31, :sunday)
    {5, 1, 7}
    
    iex> Calendar.ISO.day_of_week(2016, 10, 31, :saturday)
    {3, 1, 7}


### day_of_year(year, month, day)
*(since 1.8.0)* 
```elixir
@spec day_of_year(year(), month(), day()) :: day_of_year()
```

Calculates the day of the year from the given `year`, `month`, and `day`.

It is an integer from 1 to 366.

#### Examples

    iex> Calendar.ISO.day_of_year(2016, 1, 31)
    31
    iex> Calendar.ISO.day_of_year(-99, 2, 1)
    32
    iex> Calendar.ISO.day_of_year(2018, 2, 28)
    59


### day_rollover_relative_to_midnight_utc()
*(since 1.5.0)* 
```elixir
@spec day_rollover_relative_to_midnight_utc() :: {0, 1}
```

See `c:Calendar.day_rollover_relative_to_midnight_utc/0` for documentation.


### days_in_month(year, month)
*(since 1.4.0)* 
```elixir
@spec days_in_month(year(), month()) :: 28..31
```

Returns how many days there are in the given year-month.

#### Examples

    iex> Calendar.ISO.days_in_month(1900, 1)
    31
    iex> Calendar.ISO.days_in_month(1900, 2)
    28
    iex> Calendar.ISO.days_in_month(2000, 2)
    29
    iex> Calendar.ISO.days_in_month(2001, 2)
    28
    iex> Calendar.ISO.days_in_month(2004, 2)
    29
    iex> Calendar.ISO.days_in_month(2004, 4)
    30
    iex> Calendar.ISO.days_in_month(-1, 5)
    31


### iso_days_to_beginning_of_day(arg)
*(since 1.15.0)* 
```elixir
@spec iso_days_to_beginning_of_day(Calendar.iso_days()) :: Calendar.iso_days()
```

Converts the `t:Calendar.iso_days/0` to the first moment of the day.

#### Examples

    iex> Calendar.ISO.iso_days_to_beginning_of_day({0, {0, 86400000000}})
    {0, {0, 86400000000}}
    iex> Calendar.ISO.iso_days_to_beginning_of_day({730485, {43200000000, 86400000000}})
    {730485, {0, 86400000000}}
    iex> Calendar.ISO.iso_days_to_beginning_of_day({730485, {46800000000, 86400000000}})
    {730485, {0, 86400000000}}


### iso_days_to_end_of_day(arg)
*(since 1.15.0)* 
```elixir
@spec iso_days_to_end_of_day(Calendar.iso_days()) :: Calendar.iso_days()
```

Converts the `t:Calendar.iso_days/0` to the last moment of the day.

#### Examples

    iex> Calendar.ISO.iso_days_to_end_of_day({0, {0, 86400000000}})
    {0, {86399999999, 86400000000}}
    iex> Calendar.ISO.iso_days_to_end_of_day({730485, {43200000000, 86400000000}})
    {730485, {86399999999, 86400000000}}
    iex> Calendar.ISO.iso_days_to_end_of_day({730485, {46800000000, 86400000000}})
    {730485, {86399999999, 86400000000}}


### leap_year?(year)
*(since 1.3.0)* 
```elixir
@spec leap_year?(year()) :: boolean()
```

Returns if the given year is a leap year.

#### Examples

    iex> Calendar.ISO.leap_year?(2000)
    true
    iex> Calendar.ISO.leap_year?(2001)
    false
    iex> Calendar.ISO.leap_year?(2004)
    true
    iex> Calendar.ISO.leap_year?(1900)
    false
    iex> Calendar.ISO.leap_year?(-4)
    true


### months_in_year(year)
*(since 1.7.0)* 
```elixir
@spec months_in_year(year()) :: 12
```

Returns how many months there are in the given year.

#### Example

    iex> Calendar.ISO.months_in_year(2004)
    12


### naive_datetime_from_iso_days(arg)
*(since 1.5.0)* 
```elixir
@spec naive_datetime_from_iso_days(Calendar.iso_days()) ::
  {Calendar.year(), Calendar.month(), Calendar.day(), Calendar.hour(),
   Calendar.minute(), Calendar.second(), Calendar.microsecond()}
```

Converts the `t:Calendar.iso_days/0` format to the datetime format specified by this calendar.

#### Examples

    iex> Calendar.ISO.naive_datetime_from_iso_days({0, {0, 86400}})
    {0, 1, 1, 0, 0, 0, {0, 6}}
    iex> Calendar.ISO.naive_datetime_from_iso_days({730_485, {0, 86400}})
    {2000, 1, 1, 0, 0, 0, {0, 6}}
    iex> Calendar.ISO.naive_datetime_from_iso_days({730_485, {43200, 86400}})
    {2000, 1, 1, 12, 0, 0, {0, 6}}
    iex> Calendar.ISO.naive_datetime_from_iso_days({-365, {0, 86400000000}})
    {-1, 1, 1, 0, 0, 0, {0, 6}}


### naive_datetime_to_iso_days(year, month, day, hour, minute, second, microsecond)
*(since 1.5.0)* 
```elixir
@spec naive_datetime_to_iso_days(
  Calendar.year(),
  Calendar.month(),
  Calendar.day(),
  Calendar.hour(),
  Calendar.minute(),
  Calendar.second(),
  Calendar.microsecond()
) :: Calendar.iso_days()
```

Returns the `t:Calendar.iso_days/0` format of the specified date.

#### Examples

    iex> Calendar.ISO.naive_datetime_to_iso_days(0, 1, 1, 0, 0, 0, {0, 6})
    {0, {0, 86400000000}}
    iex> Calendar.ISO.naive_datetime_to_iso_days(2000, 1, 1, 12, 0, 0, {0, 6})
    {730485, {43200000000, 86400000000}}
    iex> Calendar.ISO.naive_datetime_to_iso_days(2000, 1, 1, 13, 0, 0, {0, 6})
    {730485, {46800000000, 86400000000}}
    iex> Calendar.ISO.naive_datetime_to_iso_days(-1, 1, 1, 0, 0, 0, {0, 6})
    {-365, {0, 86400000000}}


### naive_datetime_to_string(year, month, day, hour, minute, second, microsecond, format \\ :extended)
*(since 1.4.0)* 
```elixir
@spec naive_datetime_to_string(
  year(),
  month(),
  day(),
  Calendar.hour(),
  Calendar.minute(),
  Calendar.second(),
  Calendar.microsecond(),
  :basic | :extended
) :: String.t()
```

Converts the datetime (without time zone) into a string.

By default, returns datetimes formatted in the "extended" format,
for human readability. It also supports the "basic" format
by passing the `:basic` option.

#### Examples

    iex> Calendar.ISO.naive_datetime_to_string(2015, 2, 28, 1, 2, 3, {4, 6})
    "2015-02-28 01:02:03.000004"
    iex> Calendar.ISO.naive_datetime_to_string(2017, 8, 1, 1, 2, 3, {4, 5})
    "2017-08-01 01:02:03.00000"
    
    iex> Calendar.ISO.naive_datetime_to_string(2015, 2, 28, 1, 2, 3, {4, 6}, :basic)
    "20150228 010203.000004"


### parse_date(string)
*(since 1.10.0)* 
```elixir
@spec parse_date(String.t()) :: {:ok, {year(), month(), day()}} | {:error, atom()}
```

Parses a date `string` in the `:extended` format.

For more information on supported strings, see how this
module implements [ISO 8601](#module-iso-8601-compliance).

#### Examples

    iex> Calendar.ISO.parse_date("2015-01-23")
    {:ok, {2015, 1, 23}}
    
    iex> Calendar.ISO.parse_date("2015:01:23")
    {:error, :invalid_format}
    iex> Calendar.ISO.parse_date("2015-01-32")
    {:error, :invalid_date}


### parse_date(string, format)
*(since 1.12.0)* 
```elixir
@spec parse_date(String.t(), format()) ::
  {:ok, {year(), month(), day()}} | {:error, atom()}
```

Parses a date `string` according to a given `format`.

The `format` can either be `:basic` or `:extended`.

For more information on supported strings, see how this
module implements [ISO 8601](#module-iso-8601-compliance).

#### Examples

    iex> Calendar.ISO.parse_date("20150123", :basic)
    {:ok, {2015, 1, 23}}
    iex> Calendar.ISO.parse_date("20150123", :extended)
    {:error, :invalid_format}


### parse_duration(arg1)
*(since 1.17.0)* 
```elixir
@spec parse_duration(String.t()) :: {:ok, [Duration.unit_pair()]} | {:error, atom()}
```

Parses an ISO 8601 formatted duration string to a list of `Duration` compabitble unit pairs.

See `Duration.from_iso8601/1`.


### parse_naive_datetime(string)
*(since 1.10.0)* 
```elixir
@spec parse_naive_datetime(String.t()) ::
  {:ok, {year(), month(), day(), hour(), minute(), second(), microsecond()}}
  | {:error, atom()}
```

Parses a naive datetime `string` in the `:extended` format.

For more information on supported strings, see how this
module implements [ISO 8601](#module-iso-8601-compliance).

#### Examples

    iex> Calendar.ISO.parse_naive_datetime("2015-01-23 23:50:07")
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_naive_datetime("2015-01-23 23:50:07Z")
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_naive_datetime("2015-01-23 23:50:07-02:30")
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}}
    
    iex> Calendar.ISO.parse_naive_datetime("2015-01-23 23:50:07.0")
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 1}}}
    iex> Calendar.ISO.parse_naive_datetime("2015-01-23 23:50:07,0123456")
    {:ok, {2015, 1, 23, 23, 50, 7, {12345, 6}}}


### parse_naive_datetime(string, format)
*(since 1.12.0)* 
```elixir
@spec parse_naive_datetime(String.t(), format()) ::
  {:ok, {year(), month(), day(), hour(), minute(), second(), microsecond()}}
  | {:error, atom()}
```

Parses a naive datetime `string` according to a given `format`.

The `format` can either be `:basic` or `:extended`.

For more information on supported strings, see how this
module implements [ISO 8601](#module-iso-8601-compliance).

#### Examples

    iex> Calendar.ISO.parse_naive_datetime("20150123 235007", :basic)
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_naive_datetime("20150123 235007", :extended)
    {:error, :invalid_format}


### parse_time(string)
*(since 1.10.0)* 
```elixir
@spec parse_time(String.t()) ::
  {:ok, {hour(), minute(), second(), microsecond()}} | {:error, atom()}
```

Parses a time `string` in the `:extended` format.

For more information on supported strings, see how this
module implements [ISO 8601](#module-iso-8601-compliance).

#### Examples

    iex> Calendar.ISO.parse_time("23:50:07")
    {:ok, {23, 50, 7, {0, 0}}}
    
    iex> Calendar.ISO.parse_time("23:50:07Z")
    {:ok, {23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_time("T23:50:07Z")
    {:ok, {23, 50, 7, {0, 0}}}


### parse_time(string, format)
*(since 1.12.0)* 
```elixir
@spec parse_time(String.t(), format()) ::
  {:ok, {hour(), minute(), second(), microsecond()}} | {:error, atom()}
```

Parses a time `string` according to a given `format`.

The `format` can either be `:basic` or `:extended`.

For more information on supported strings, see how this
module implements [ISO 8601](#module-iso-8601-compliance).

#### Examples

    iex> Calendar.ISO.parse_time("235007", :basic)
    {:ok, {23, 50, 7, {0, 0}}}
    iex> Calendar.ISO.parse_time("235007", :extended)
    {:error, :invalid_format}


### parse_utc_datetime(string)
*(since 1.10.0)* 
```elixir
@spec parse_utc_datetime(String.t()) ::
  {:ok, {year(), month(), day(), hour(), minute(), second(), microsecond()},
   utc_offset()}
  | {:error, atom()}
```

Parses a UTC datetime `string` in the `:extended` format.

For more information on supported strings, see how this
module implements [ISO 8601](#module-iso-8601-compliance).

#### Examples

    iex> Calendar.ISO.parse_utc_datetime("2015-01-23 23:50:07Z")
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}, 0}
    
    iex> Calendar.ISO.parse_utc_datetime("2015-01-23 23:50:07+02:30")
    {:ok, {2015, 1, 23, 21, 20, 7, {0, 0}}, 9000}
    
    iex> Calendar.ISO.parse_utc_datetime("2015-01-23 23:50:07")
    {:error, :missing_offset}


### parse_utc_datetime(string, format)
*(since 1.12.0)* 
```elixir
@spec parse_utc_datetime(String.t(), format()) ::
  {:ok, {year(), month(), day(), hour(), minute(), second(), microsecond()},
   utc_offset()}
  | {:error, atom()}
```

Parses a UTC datetime `string` according to a given `format`.

The `format` can either be `:basic` or `:extended`.

For more information on supported strings, see how this
module implements [ISO 8601](#module-iso-8601-compliance).

#### Examples

    iex> Calendar.ISO.parse_utc_datetime("20150123 235007Z", :basic)
    {:ok, {2015, 1, 23, 23, 50, 7, {0, 0}}, 0}
    iex> Calendar.ISO.parse_utc_datetime("20150123 235007Z", :extended)
    {:error, :invalid_format}


### quarter_of_year(year, month, day)
*(since 1.8.0)* 
```elixir
@spec quarter_of_year(year(), month(), day()) :: quarter_of_year()
```

Calculates the quarter of the year from the given `year`, `month`, and `day`.

It is an integer from 1 to 4.

#### Examples

    iex> Calendar.ISO.quarter_of_year(2016, 1, 31)
    1
    iex> Calendar.ISO.quarter_of_year(2016, 4, 3)
    2
    iex> Calendar.ISO.quarter_of_year(-99, 9, 31)
    3
    iex> Calendar.ISO.quarter_of_year(2018, 12, 28)
    4


### shift_date(year, month, day, duration)

```elixir
@spec shift_date(year(), month(), day(), Duration.t()) :: {year(), month(), day()}
```

Shifts Date by Duration according to its calendar.

#### Examples

    iex> Calendar.ISO.shift_date(2016, 1, 3, Duration.new!(month: 2))
    {2016, 3, 3}
    iex> Calendar.ISO.shift_date(2016, 2, 29, Duration.new!(month: 1))
    {2016, 3, 29}
    iex> Calendar.ISO.shift_date(2016, 1, 31, Duration.new!(month: 1))
    {2016, 2, 29}
    iex> Calendar.ISO.shift_date(2016, 1, 31, Duration.new!(year: 4, day: 1))
    {2020, 2, 1}


### shift_naive_datetime(year, month, day, hour, minute, second, microsecond, duration)

```elixir
@spec shift_naive_datetime(
  year(),
  month(),
  day(),
  hour(),
  minute(),
  second(),
  microsecond(),
  Duration.t()
) :: {year(), month(), day(), hour(), minute(), second(), microsecond()}
```

Shifts NaiveDateTime by Duration according to its calendar.

#### Examples

    iex> Calendar.ISO.shift_naive_datetime(2016, 1, 3, 0, 0, 0, {0, 0}, Duration.new!(hour: 1))
    {2016, 1, 3, 1, 0, 0, {0, 0}}
    iex> Calendar.ISO.shift_naive_datetime(2016, 1, 3, 0, 0, 0, {0, 0}, Duration.new!(hour: 30))
    {2016, 1, 4, 6, 0, 0, {0, 0}}
    iex> Calendar.ISO.shift_naive_datetime(2016, 1, 3, 0, 0, 0, {0, 0}, Duration.new!(microsecond: {100, 6}))
    {2016, 1, 3, 0, 0, 0, {100, 6}}


### shift_time(hour, minute, second, microsecond, duration)

```elixir
@spec shift_time(hour(), minute(), second(), microsecond(), Duration.t()) ::
  {hour(), minute(), second(), microsecond()}
```

Shifts Time by Duration units according to its calendar.

#### Examples

    iex> Calendar.ISO.shift_time(13, 0, 0, {0, 0}, Duration.new!(hour: 2))
    {15, 0, 0, {0, 0}}
    iex> Calendar.ISO.shift_time(13, 0, 0, {0, 0}, Duration.new!(microsecond: {100, 6}))
    {13, 0, 0, {100, 6}}


### time_from_day_fraction(arg)
*(since 1.5.0)* 
```elixir
@spec time_from_day_fraction(Calendar.day_fraction()) ::
  {hour(), minute(), second(), microsecond()}
```

Converts a day fraction to this Calendar's representation of time.

#### Examples

    iex> Calendar.ISO.time_from_day_fraction({1, 2})
    {12, 0, 0, {0, 6}}
    iex> Calendar.ISO.time_from_day_fraction({13, 24})
    {13, 0, 0, {0, 6}}


### time_to_day_fraction(hour, minute, second, arg)
*(since 1.5.0)* 
```elixir
@spec time_to_day_fraction(
  Calendar.hour(),
  Calendar.minute(),
  Calendar.second(),
  Calendar.microsecond()
) :: Calendar.day_fraction()
```

Returns the normalized day fraction of the specified time.

#### Examples

    iex> Calendar.ISO.time_to_day_fraction(0, 0, 0, {0, 6})
    {0, 86400000000}
    iex> Calendar.ISO.time_to_day_fraction(12, 34, 56, {123, 6})
    {45296000123, 86400000000}


### time_to_string(hour, minute, second, microsecond, format \\ :extended)
*(since 1.5.0)* 
```elixir
@spec time_to_string(
  Calendar.hour(),
  Calendar.minute(),
  Calendar.second(),
  Calendar.microsecond(),
  :basic | :extended
) :: String.t()
```

Converts the given time into a string.

By default, returns times formatted in the "extended" format,
for human readability. It also supports the "basic" format
by passing the `:basic` option.

#### Examples

    iex> Calendar.ISO.time_to_string(2, 2, 2, {2, 6})
    "02:02:02.000002"
    iex> Calendar.ISO.time_to_string(2, 2, 2, {2, 2})
    "02:02:02.00"
    iex> Calendar.ISO.time_to_string(2, 2, 2, {2, 0})
    "02:02:02"
    
    iex> Calendar.ISO.time_to_string(2, 2, 2, {2, 6}, :basic)
    "020202.000002"
    iex> Calendar.ISO.time_to_string(2, 2, 2, {2, 6}, :extended)
    "02:02:02.000002"


### time_unit_to_precision(int)
*(since 1.15.0)* 
```elixir
@spec time_unit_to_precision(System.time_unit()) :: 0..6
```

Converts a `t:System.time_unit/0` to precision.

Integer-based time units always get maximum precision.

#### Examples

    iex> Calendar.ISO.time_unit_to_precision(:nanosecond)
    6
    
    iex> Calendar.ISO.time_unit_to_precision(:second)
    0
    
    iex> Calendar.ISO.time_unit_to_precision(1)
    6


### valid_date?(year, month, day)
*(since 1.5.0)* 
```elixir
@spec valid_date?(year(), month(), day()) :: boolean()
```

Determines if the date given is valid according to the proleptic Gregorian calendar.

#### Examples

    iex> Calendar.ISO.valid_date?(2015, 2, 28)
    true
    iex> Calendar.ISO.valid_date?(2015, 2, 30)
    false
    iex> Calendar.ISO.valid_date?(-1, 12, 31)
    true
    iex> Calendar.ISO.valid_date?(-1, 12, 32)
    false


### valid_time?(hour, minute, second, microsecond)
*(since 1.5.0)* 
```elixir
@spec valid_time?(
  Calendar.hour(),
  Calendar.minute(),
  Calendar.second(),
  Calendar.microsecond()
) ::
  boolean()
```

Determines if the date given is valid according to the proleptic Gregorian calendar.

Leap seconds are not supported by the built-in Calendar.ISO.

#### Examples

    iex> Calendar.ISO.valid_time?(10, 50, 25, {3006, 6})
    true
    iex> Calendar.ISO.valid_time?(23, 59, 60, {0, 0})
    false
    iex> Calendar.ISO.valid_time?(24, 0, 0, {0, 0})
    false


### year_of_era(year)
*(since 1.8.0)* 
```elixir
@spec year_of_era(year()) :: {1..10000, era()}
```

Calculates the year and era from the given `year`.

The ISO calendar has two eras: the "current era" (CE) which
starts in year `1` and is defined as era `1`. And "before the current
era" (BCE) for those years less than `1`, defined as era `0`.

#### Examples

    iex> Calendar.ISO.year_of_era(1)
    {1, 1}
    iex> Calendar.ISO.year_of_era(2018)
    {2018, 1}
    iex> Calendar.ISO.year_of_era(0)
    {1, 0}
    iex> Calendar.ISO.year_of_era(-1)
    {2, 0}


### year_of_era(year, month, day)
*(since 1.13.0)* 
```elixir
@spec year_of_era(year(), month(), day()) :: {1..10000, era()}
```

Calendar callback to compute the year and era from the
given `year`, `month` and `day`.

In the ISO calendar, the new year coincides with the new era,
so the `month` and `day` arguments are discarded. If you only
have the year available, you can `year_of_era/1` instead.

#### Examples

    iex> Calendar.ISO.year_of_era(1, 1, 1)
    {1, 1}
    iex> Calendar.ISO.year_of_era(2018, 12, 1)
    {2018, 1}
    iex> Calendar.ISO.year_of_era(0, 1, 1)
    {1, 0}
    iex> Calendar.ISO.year_of_era(-1, 12, 1)
    {2, 0}




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
