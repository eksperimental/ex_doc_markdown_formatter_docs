# Calendar behaviour
(Elixir v1.18.0-dev)

This module defines the responsibilities for working with
calendars, dates, times and datetimes in Elixir.

It defines types and the minimal implementation
for a calendar behaviour in Elixir. The goal of the calendar
features in Elixir is to provide a base for interoperability
rather than a full-featured datetime API.

For the actual date, time and datetime structs, see `Date`,
`Time`, `NaiveDateTime`, and `DateTime`.

Types for year, month, day, and more are *overspecified*.
For example, the `t:month/0` type is specified as an integer
instead of `1..12`. This is because different calendars may
have a different number of days per month.


## Types

### calendar()

```elixir
@type calendar() :: module()
```

A calendar implementation.


### date()

```elixir
@type date() :: %{
  optional(any()) =&gt; any(),
  calendar: calendar(),
  year: year(),
  month: month(),
  day: day()
}
```

Any map or struct that contains the date fields.


### datetime()

```elixir
@type datetime() :: %{
  optional(any()) =&gt; any(),
  calendar: calendar(),
  year: year(),
  month: month(),
  day: day(),
  hour: hour(),
  minute: minute(),
  second: second(),
  microsecond: microsecond(),
  time_zone: time_zone(),
  zone_abbr: zone_abbr(),
  utc_offset: utc_offset(),
  std_offset: std_offset()
}
```

Any map or struct that contains the datetime fields.


### day()

```elixir
@type day() :: pos_integer()
```



### day_fraction()

```elixir
@type day_fraction() ::
  {parts_in_day :: non_neg_integer(), parts_per_day :: pos_integer()}
```

The internal time format is used when converting between calendars.

It represents time as a fraction of a day (starting from midnight).
`parts_in_day` specifies how much of the day is already passed,
while `parts_per_day` signifies how many parts are there in a day.


### day_of_era()

```elixir
@type day_of_era() :: {day :: non_neg_integer(), era()}
```

A tuple representing the `day` and the `era`.


### day_of_week()

```elixir
@type day_of_week() :: non_neg_integer()
```



### era()

```elixir
@type era() :: non_neg_integer()
```



### hour()

```elixir
@type hour() :: non_neg_integer()
```



### iso_days()

```elixir
@type iso_days() :: {days :: integer(), day_fraction()}
```

The internal date format that is used when converting between calendars.

This is the number of days including the fractional part that has passed of
the last day since `0000-01-01+00:00T00:00.000000` in ISO 8601 notation (also
known as *midnight 1 January BC 1* of the proleptic Gregorian calendar).


### microsecond()

```elixir
@type microsecond() :: {value :: non_neg_integer(), precision :: non_neg_integer()}
```

Microseconds with stored precision.

The precision represents the number of digits that must be used when
representing the microseconds to external format. If the precision is `0`,
it means microseconds must be skipped.


### minute()

```elixir
@type minute() :: non_neg_integer()
```



### month()

```elixir
@type month() :: pos_integer()
```



### naive_datetime()

```elixir
@type naive_datetime() :: %{
  optional(any()) =&gt; any(),
  calendar: calendar(),
  year: year(),
  month: month(),
  day: day(),
  hour: hour(),
  minute: minute(),
  second: second(),
  microsecond: microsecond()
}
```

Any map or struct that contains the naive datetime fields.


### second()

```elixir
@type second() :: non_neg_integer()
```



### std_offset()

```elixir
@type std_offset() :: integer()
```

The time zone standard offset in ISO seconds (typically not zero in summer times).

It must be added to `t:utc_offset/0` to get the total offset from UTC used for "wall time".


### time()

```elixir
@type time() :: %{
  optional(any()) =&gt; any(),
  hour: hour(),
  minute: minute(),
  second: second(),
  microsecond: microsecond()
}
```

Any map or struct that contains the time fields.


### time_zone()

```elixir
@type time_zone() :: String.t()
```

The time zone ID according to the IANA tz database (for example, `Europe/Zurich`).


### time_zone_database()

```elixir
@type time_zone_database() :: module()
```

Specifies the time zone database for calendar operations.

Many functions in the `DateTime` module require a time zone database.
By default, this module uses the default time zone database returned by
`Calendar.get_time_zone_database/0`, which defaults to
`Calendar.UTCOnlyTimeZoneDatabase`. This database only handles `Etc/UTC`
datetimes and returns `{:error, :utc_only_time_zone_database}`
for any other time zone.

Other time zone databases (including ones provided by packages)
can be configured as default either via configuration:

    config :elixir, :time_zone_database, CustomTimeZoneDatabase

or by calling `Calendar.put_time_zone_database/1`.

See `Calendar.TimeZoneDatabase` for more information on custom
time zone databases.


### utc_offset()

```elixir
@type utc_offset() :: integer()
```

The time zone UTC offset in ISO seconds for standard time.

See also `t:std_offset/0`.


### week()

```elixir
@type week() :: pos_integer()
```



### year()

```elixir
@type year() :: integer()
```



### zone_abbr()

```elixir
@type zone_abbr() :: String.t()
```

The time zone abbreviation (for example, `CET` or `CEST` or `BST`).


## Callbacks

### date_to_string(year, month, day)

```elixir
@callback date_to_string(year(), month(), day()) :: String.t()
```

Converts the date into a string according to the calendar.


### datetime_to_string(
  year,
  month,
  day,
  hour,
  minute,
  second,
  microsecond,
  time_zone,
  zone_abbr,
  utc_offset,
  std_offset
)

```elixir
@callback datetime_to_string(
  year(),
  month(),
  day(),
  hour(),
  minute(),
  second(),
  microsecond(),
  time_zone(),
  zone_abbr(),
  utc_offset(),
  std_offset()
) :: String.t()
```

Converts the datetime (with time zone) into a string according to the calendar.


### day_of_era(year, month, day)

```elixir
@callback day_of_era(year(), month(), day()) :: day_of_era()
```

Calculates the day and era from the given `year`, `month`, and `day`.


### day_of_week(year, month, day, starting_on)

```elixir
@callback day_of_week(year(), month(), day(), starting_on :: :default | atom()) ::
  {day_of_week(), first_day_of_week :: non_neg_integer(),
   last_day_of_week :: non_neg_integer()}
```

Calculates the day of the week from the given `year`, `month`, and `day`.

`starting_on` represents the starting day of the week. All
calendars must support at least the `:default` value. They may
also support other values representing their days of the week.


### day_of_year(year, month, day)

```elixir
@callback day_of_year(year(), month(), day()) :: non_neg_integer()
```

Calculates the day of the year from the given `year`, `month`, and `day`.


### day_rollover_relative_to_midnight_utc()

```elixir
@callback day_rollover_relative_to_midnight_utc() :: day_fraction()
```

Define the rollover moment for the calendar.

This is the moment, in your calendar, when the current day ends
and the next day starts.

The result of this function is used to check if two calendars roll over at
the same time of day. If they do not, we can only convert datetimes and times
between them. If they do, this means that we can also convert dates as well
as naive datetimes between them.

This day fraction should be in its most simplified form possible, to make comparisons fast.

#### Examples

- If in your calendar a new day starts at midnight, return `{0, 1}`.
- If in your calendar a new day starts at sunrise, return `{1, 4}`.
- If in your calendar a new day starts at noon, return `{1, 2}`.
- If in your calendar a new day starts at sunset, return `{3, 4}`.


### days_in_month(year, month)

```elixir
@callback days_in_month(year(), month()) :: day()
```

Returns how many days there are in the given month of the given year.


### iso_days_to_beginning_of_day(iso_days)
*(since 1.15.0)* 
```elixir
@callback iso_days_to_beginning_of_day(iso_days()) :: iso_days()
```

Converts the given `t:iso_days/0` to the first moment of the day.


### iso_days_to_end_of_day(iso_days)
*(since 1.15.0)* 
```elixir
@callback iso_days_to_end_of_day(iso_days()) :: iso_days()
```

Converts the given `t:iso_days/0` to the last moment of the day.


### leap_year?(year)

```elixir
@callback leap_year?(year()) :: boolean()
```

Returns `true` if the given year is a leap year.

A leap year is a year of a longer length than normal. The exact meaning
is up to the calendar. A calendar must return `false` if it does not support
the concept of leap years.


### months_in_year(year)

```elixir
@callback months_in_year(year()) :: month()
```

Returns how many months there are in the given year.


### naive_datetime_from_iso_days(iso_days)

```elixir
@callback naive_datetime_from_iso_days(iso_days()) ::
  {year(), month(), day(), hour(), minute(), second(), microsecond()}
```

Converts `t:iso_days/0` to the calendar's datetime format.


### naive_datetime_to_iso_days(year, month, day, hour, minute, second, microsecond)

```elixir
@callback naive_datetime_to_iso_days(
  year(),
  month(),
  day(),
  hour(),
  minute(),
  second(),
  microsecond()
) ::
  iso_days()
```

Converts the datetime (without time zone) into the `t:iso_days/0` format.


### naive_datetime_to_string(year, month, day, hour, minute, second, microsecond)

```elixir
@callback naive_datetime_to_string(
  year(),
  month(),
  day(),
  hour(),
  minute(),
  second(),
  microsecond()
) ::
  String.t()
```

Converts the naive datetime (without time zone) into a string according to the calendar.


### parse_date(t)
*(since 1.10.0)* 
```elixir
@callback parse_date(String.t()) :: {:ok, {year(), month(), day()}} | {:error, atom()}
```

Parses the string representation for a date returned by `c:date_to_string/3`
into a date tuple.


### parse_naive_datetime(t)
*(since 1.10.0)* 
```elixir
@callback parse_naive_datetime(String.t()) ::
  {:ok, {year(), month(), day(), hour(), minute(), second(), microsecond()}}
  | {:error, atom()}
```

Parses the string representation for a naive datetime returned by
`c:naive_datetime_to_string/7` into a naive datetime tuple.

The given string may contain a timezone offset but it is ignored.


### parse_time(t)
*(since 1.10.0)* 
```elixir
@callback parse_time(String.t()) ::
  {:ok, {hour(), minute(), second(), microsecond()}} | {:error, atom()}
```

Parses the string representation for a time returned by `c:time_to_string/4`
into a time tuple.


### parse_utc_datetime(t)
*(since 1.10.0)* 
```elixir
@callback parse_utc_datetime(String.t()) ::
  {:ok, {year(), month(), day(), hour(), minute(), second(), microsecond()},
   utc_offset()}
  | {:error, atom()}
```

Parses the string representation for a datetime returned by
`c:datetime_to_string/11` into a datetime tuple.

The returned datetime must be in UTC. The original `utc_offset`
it was written in must be returned in the result.


### quarter_of_year(year, month, day)

```elixir
@callback quarter_of_year(year(), month(), day()) :: non_neg_integer()
```

Calculates the quarter of the year from the given `year`, `month`, and `day`.


### shift_date(year, month, day, t)
*(since 1.17.0)* 
```elixir
@callback shift_date(year(), month(), day(), Duration.t()) :: {year(), month(), day()}
```

Shifts date by given duration according to its calendar.


### shift_naive_datetime(year, month, day, hour, minute, second, microsecond, t)
*(since 1.17.0)* 
```elixir
@callback shift_naive_datetime(
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

Shifts naive datetime by given duration according to its calendar.


### shift_time(hour, minute, second, microsecond, t)
*(since 1.17.0)* 
```elixir
@callback shift_time(hour(), minute(), second(), microsecond(), Duration.t()) ::
  {hour(), minute(), second(), microsecond()}
```

Shifts time by given duration according to its calendar.


### time_from_day_fraction(day_fraction)

```elixir
@callback time_from_day_fraction(day_fraction()) ::
  {hour(), minute(), second(), microsecond()}
```

Converts `t:day_fraction/0` to the calendar's time format.


### time_to_day_fraction(hour, minute, second, microsecond)

```elixir
@callback time_to_day_fraction(hour(), minute(), second(), microsecond()) ::
  day_fraction()
```

Converts the given time to the `t:day_fraction/0` format.


### time_to_string(hour, minute, second, microsecond)

```elixir
@callback time_to_string(hour(), minute(), second(), microsecond()) :: String.t()
```

Converts the time into a string according to the calendar.


### valid_date?(year, month, day)

```elixir
@callback valid_date?(year(), month(), day()) :: boolean()
```

Should return `true` if the given date describes a proper date in the calendar.


### valid_time?(hour, minute, second, microsecond)

```elixir
@callback valid_time?(hour(), minute(), second(), microsecond()) :: boolean()
```

Should return `true` if the given time describes a proper time in the calendar.


### year_of_era(year, month, day)

```elixir
@callback year_of_era(year(), month(), day()) :: {year(), era()}
```

Calculates the year and era from the given `year`.


## Functions

### compatible_calendars?(calendar, calendar)
*(since 1.5.0)* 
```elixir
@spec compatible_calendars?(calendar(), calendar()) :: boolean()
```

Returns `true` if two calendars have the same moment of starting a new day,
`false` otherwise.

If two calendars are not compatible, we can only convert datetimes and times
between them. If they are compatible, this means that we can also convert
dates as well as naive datetimes between them.


### get_time_zone_database()
*(since 1.8.0)* 
```elixir
@spec get_time_zone_database() :: time_zone_database()
```

Gets the current time zone database.


### put_time_zone_database(database)
*(since 1.8.0)* 
```elixir
@spec put_time_zone_database(time_zone_database()) :: :ok
```

Sets the current time zone database.


### strftime(date_or_time_or_datetime, string_format, user_options \\ [])
*(since 1.11.0)* 
```elixir
@spec strftime(map(), String.t(), keyword()) :: String.t()
```

Formats the given date, time, or datetime into a string.

The datetime can be any of the `Calendar` types (`Time`, `Date`,
`NaiveDateTime`, and `DateTime`) or any map, as long as they
contain all of the relevant fields necessary for formatting.
For example, if you use `%Y` to format the year, the datetime
must have the `:year` field. Therefore, if you pass a `Time`,
or a map without the `:year` field to a format that expects `%Y`,
an error will be raised.

Examples of common usage:

    iex> Calendar.strftime(~U[2019-08-26 13:52:06.0Z], "%y-%m-%d %I:%M:%S %p")
    "19-08-26 01:52:06 PM"
    
    iex> Calendar.strftime(~U[2019-08-26 13:52:06.0Z], "%a, %B %d %Y")
    "Mon, August 26 2019"

#### User Options

- `:preferred_datetime` - a string for the preferred format to show datetimes,
  it can't contain the `%c` format and defaults to `"%Y-%m-%d %H:%M:%S"`
  if the option is not received

- `:preferred_date` - a string for the preferred format to show dates,
  it can't contain the `%x` format and defaults to `"%Y-%m-%d"`
  if the option is not received

- `:preferred_time` - a string for the preferred format to show times,
  it can't contain the `%X` format and defaults to `"%H:%M:%S"`
  if the option is not received

- `:am_pm_names` - a function that receives either `:am` or `:pm` and returns
  the name of the period of the day, if the option is not received it defaults
  to a function that returns `"am"` and `"pm"`, respectively

- `:month_names` - a function that receives a number and returns the name of
  the corresponding month, if the option is not received it defaults to a
  function that returns the month names in English

- `:abbreviated_month_names` - a function that receives a number and returns the
  abbreviated name of the corresponding month, if the option is not received it
  defaults to a function that returns the abbreviated month names in English

- `:day_of_week_names` - a function that receives a number and returns the name of
  the corresponding day of week, if the option is not received it defaults to a
  function that returns the day of week names in English

- `:abbreviated_day_of_week_names` - a function that receives a number and returns
  the abbreviated name of the corresponding day of week, if the option is not received
  it defaults to a function that returns the abbreviated day of week names in English

#### Formatting syntax

The formatting syntax for the `string_format` argument is a sequence of characters in
the following format:

    %<padding><width><format>

where:

- `%`: indicates the start of a formatted section
- `<padding>`: set the padding (see below)
- `<width>`: a number indicating the minimum size of the formatted section
- `<format>`: the format itself (see below)

##### Accepted padding options

- `-`: no padding, removes all padding from the format
- `_`: pad with spaces
- `0`: pad with zeroes

##### Accepted string formats

The accepted formats for `string_format` are:

Format | Description                                                             | Examples (in ISO)
:----- | :-----------------------------------------------------------------------| :------------------------
a      | Abbreviated name of day                                                 | Mon
A      | Full name of day                                                        | Monday
b      | Abbreviated month name                                                  | Jan
B      | Full month name                                                         | January
c      | Preferred date+time representation                                      | 2018-10-17 12:34:56
d      | Day of the month                                                        | 01, 31
f      | Microseconds *(does not support width and padding modifiers)*           | 000000, 999999, 0123
H      | Hour using a 24-hour clock                                              | 00, 23
I      | Hour using a 12-hour clock                                              | 01, 12
j      | Day of the year                                                         | 001, 366
m      | Month                                                                   | 01, 12
M      | Minute                                                                  | 00, 59
p      | "AM" or "PM" (noon is "PM", midnight as "AM")                           | AM, PM
P      | "am" or "pm" (noon is "pm", midnight as "am")                           | am, pm
q      | Quarter                                                                 | 1, 2, 3, 4
s      | Number of seconds since the Epoch, 1970-01-01 00:00:00+0000 (UTC)       | 1565888877
S      | Second                                                                  | 00, 59, 60
u      | Day of the week                                                         | 1 (Monday), 7 (Sunday)
x      | Preferred date (without time) representation                            | 2018-10-17
X      | Preferred time (without date) representation                            | 12:34:56
y      | Year as 2-digits                                                        | 01, 01, 86, 18
Y      | Year                                                                    | -0001, 0001, 1986
z      | +hhmm/-hhmm time zone offset from UTC (empty string if naive)           | +0300, -0530
Z      | Time zone abbreviation (empty string if naive)                          | CET, BRST
%      | Literal "%" character                                                   | %

Any other character will be interpreted as an invalid format and raise an error.

#### Examples

Without user options:

    iex> Calendar.strftime(~U[2019-08-26 13:52:06.0Z], "%y-%m-%d %I:%M:%S %p")
    "19-08-26 01:52:06 PM"
    
    iex> Calendar.strftime(~U[2019-08-26 13:52:06.0Z], "%a, %B %d %Y")
    "Mon, August 26 2019"
    
    iex> Calendar.strftime(~U[2020-04-02 13:52:06.0Z], "%B %-d, %Y")
    "April 2, 2020"
    
    iex> Calendar.strftime(~U[2019-08-26 13:52:06.0Z], "%c")
    "2019-08-26 13:52:06"

With user options:

    iex> Calendar.strftime(~U[2019-08-26 13:52:06.0Z], "%c", preferred_datetime: "%H:%M:%S %d-%m-%y")
    "13:52:06 26-08-19"
    
    iex> Calendar.strftime(
    ...>  ~U[2019-08-26 13:52:06.0Z],
    ...>  "%A",
    ...>  day_of_week_names: fn day_of_week ->
    ...>    {"segunda-feira", "terça-feira", "quarta-feira", "quinta-feira",
    ...>    "sexta-feira", "sábado", "domingo"}
    ...>    |> elem(day_of_week - 1)
    ...>  end
    ...>)
    "segunda-feira"
    
    iex> Calendar.strftime(
    ...>  ~U[2019-08-26 13:52:06.0Z],
    ...>  "%B",
    ...>  month_names: fn month ->
    ...>    {"січень", "лютий", "березень", "квітень", "травень", "червень",
    ...>    "липень", "серпень", "вересень", "жовтень", "листопад", "грудень"}
    ...>    |> elem(month - 1)
    ...>  end
    ...>)
    "серпень"


### truncate(microsecond_tuple, atom)
*(since 1.6.0)* 
```elixir
@spec truncate(microsecond(), :microsecond | :millisecond | :second) :: microsecond()
```

Returns a microsecond tuple truncated to a given precision (`:microsecond`,
`:millisecond`, or `:second`).




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
