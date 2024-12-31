# IO.ANSI 
(Elixir v1.18.0-dev)

Functionality to render ANSI escape sequences.

[ANSI escape sequences](https://en.wikipedia.org/wiki/ANSI_escape_code)
are characters embedded in text used to control formatting, color, and
other output options on video text terminals.

ANSI escapes are typically enabled on all Unix terminals. They are also
available on Windows consoles from Windows 10, although it must be
explicitly enabled for the current user in the registry by running the
following command:

    reg add HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1

After running the command above, you must restart your current console.

## Examples

Because the ANSI escape sequences are embedded in text, the normal usage of
these functions is to concatenate their output with text.

    formatted_text = IO.ANSI.blue_background() <> "Example" <> IO.ANSI.reset()
    IO.puts(formatted_text)

A higher level and more convenient API is also available via `IO.ANSI.format/1`,
where you use atoms to represent each ANSI escape sequence and by default
checks if ANSI is enabled:

    IO.puts(IO.ANSI.format([:blue_background, "Example"]))

In case ANSI is disabled, the ANSI escape sequences are simply discarded.


## Types

### ansicode()

```elixir
@type ansicode() :: atom()
```



### ansidata()

```elixir
@type ansidata() :: ansilist() | ansicode() | binary()
```



### ansilist()

```elixir
@type ansilist() ::
  maybe_improper_list(
    char() | ansicode() | binary() | ansilist(),
    binary() | ansicode() | []
  )
```



## Functions

### black()

```elixir
@spec black() :: String.t()
```

Sets foreground color to black.


### black_background()

```elixir
@spec black_background() :: String.t()
```

Sets background color to black.


### blink_off()

```elixir
@spec blink_off() :: String.t()
```

Blink: off.


### blink_rapid()

```elixir
@spec blink_rapid() :: String.t()
```

Blink: rapid. MS-DOS ANSI.SYS; 150 per minute or more; not widely supported.


### blink_slow()

```elixir
@spec blink_slow() :: String.t()
```

Blink: slow. Less than 150 per minute.


### blue()

```elixir
@spec blue() :: String.t()
```

Sets foreground color to blue.


### blue_background()

```elixir
@spec blue_background() :: String.t()
```

Sets background color to blue.


### bright()

```elixir
@spec bright() :: String.t()
```

Bright (increased intensity) or bold.


### clear()

```elixir
@spec clear() :: String.t()
```

Clears screen.


### clear_line()

```elixir
@spec clear_line() :: String.t()
```

Clears line.


### color(code)

```elixir
@spec color(0..255) :: String.t()
```

Sets foreground color.


### color(r, g, b)

```elixir
@spec color(0..5, 0..5, 0..5) :: String.t()
```

Sets the foreground color from individual RGB values.

Valid values for each color are in the range 0 to 5.


### color_background(code)

```elixir
@spec color_background(0..255) :: String.t()
```

Sets background color.


### color_background(r, g, b)

```elixir
@spec color_background(0..5, 0..5, 0..5) :: String.t()
```

Sets the background color from individual RGB values.

Valid values for each color are in the range 0 to 5.


### conceal()

```elixir
@spec conceal() :: String.t()
```

Conceal. Not widely supported.


### crossed_out()

```elixir
@spec crossed_out() :: String.t()
```

Crossed-out. Characters legible, but marked for deletion. Not widely supported.


### cursor(line, column)

```elixir
@spec cursor(non_neg_integer(), non_neg_integer()) :: String.t()
```

Sends cursor to the absolute position specified by `line` and `column`.

Line `0` and column `0` would mean the top left corner.


### cursor_down(lines \\ 1)

```elixir
@spec cursor_down(pos_integer()) :: String.t()
```

Sends cursor `lines` down.


### cursor_left(columns \\ 1)

```elixir
@spec cursor_left(pos_integer()) :: String.t()
```

Sends cursor `columns` to the left.


### cursor_right(columns \\ 1)

```elixir
@spec cursor_right(pos_integer()) :: String.t()
```

Sends cursor `columns` to the right.


### cursor_up(lines \\ 1)

```elixir
@spec cursor_up(pos_integer()) :: String.t()
```

Sends cursor `lines` up.


### cyan()

```elixir
@spec cyan() :: String.t()
```

Sets foreground color to cyan.


### cyan_background()

```elixir
@spec cyan_background() :: String.t()
```

Sets background color to cyan.


### default_background()

```elixir
@spec default_background() :: String.t()
```

Default background color.


### default_color()

```elixir
@spec default_color() :: String.t()
```

Default text color.


### enabled?()

```elixir
@spec enabled?() :: boolean()
```

Checks if ANSI coloring is supported and enabled on this machine.

This function simply reads the configuration value for
`:ansi_enabled` in the `:elixir` application. The value is by
default `false` unless Elixir can detect during startup that
both `stdout` and `stderr` are terminals.


### encircled()

```elixir
@spec encircled() :: String.t()
```

Encircled.


### faint()

```elixir
@spec faint() :: String.t()
```

Faint (decreased intensity). Not widely supported.


### font_1()

```elixir
@spec font_1() :: String.t()
```

Sets alternative font 1.


### font_2()

```elixir
@spec font_2() :: String.t()
```

Sets alternative font 2.


### font_3()

```elixir
@spec font_3() :: String.t()
```

Sets alternative font 3.


### font_4()

```elixir
@spec font_4() :: String.t()
```

Sets alternative font 4.


### font_5()

```elixir
@spec font_5() :: String.t()
```

Sets alternative font 5.


### font_6()

```elixir
@spec font_6() :: String.t()
```

Sets alternative font 6.


### font_7()

```elixir
@spec font_7() :: String.t()
```

Sets alternative font 7.


### font_8()

```elixir
@spec font_8() :: String.t()
```

Sets alternative font 8.


### font_9()

```elixir
@spec font_9() :: String.t()
```

Sets alternative font 9.


### format(ansidata, emit? \\ enabled?())

```elixir
@spec format(ansidata(), boolean()) :: IO.chardata()
```

Formats a chardata-like argument by converting named ANSI sequences into actual
ANSI codes.

The named sequences are represented by atoms.

It will also append an `IO.ANSI.reset/0` to the chardata when a conversion is
performed. If you don't want this behavior, use `format_fragment/2`.

An optional boolean parameter can be passed to enable or disable
emitting actual ANSI codes. When `false`, no ANSI codes will be emitted.
By default checks if ANSI is enabled using the `enabled?/0` function.

An `ArgumentError` will be raised if an invalid ANSI code is provided.

#### Examples

    iex> IO.ANSI.format(["Hello, ", :red, :bright, "world!"], true)
    [[[[[[], "Hello, "] | "\e[31m"] | "\e[1m"], "world!"] | "\e[0m"]


### format_fragment(ansidata, emit? \\ enabled?())

```elixir
@spec format_fragment(ansidata(), boolean()) :: IO.chardata()
```

Formats a chardata-like argument by converting named ANSI sequences into actual
ANSI codes.

The named sequences are represented by atoms.

An optional boolean parameter can be passed to enable or disable
emitting actual ANSI codes. When `false`, no ANSI codes will be emitted.
By default checks if ANSI is enabled using the `enabled?/0` function.

#### Examples

    iex> IO.ANSI.format_fragment([:bright, ~c"Word"], true)
    [[[[[[] | "\e[1m"], 87], 111], 114], 100]


### framed()

```elixir
@spec framed() :: String.t()
```

Framed.


### green()

```elixir
@spec green() :: String.t()
```

Sets foreground color to green.


### green_background()

```elixir
@spec green_background() :: String.t()
```

Sets background color to green.


### home()

```elixir
@spec home() :: String.t()
```

Sends cursor home.


### inverse()

```elixir
@spec inverse() :: String.t()
```

Image: negative. Swap foreground and background.


### inverse_off()

```elixir
@spec inverse_off() :: String.t()
```

Image: positive. Normal foreground and background.


### italic()

```elixir
@spec italic() :: String.t()
```

Italic: on. Not widely supported. Sometimes treated as inverse.


### light_black()

```elixir
@spec light_black() :: String.t()
```

Sets foreground color to light black.


### light_black_background()

```elixir
@spec light_black_background() :: String.t()
```

Sets background color to light black.


### light_blue()

```elixir
@spec light_blue() :: String.t()
```

Sets foreground color to light blue.


### light_blue_background()

```elixir
@spec light_blue_background() :: String.t()
```

Sets background color to light blue.


### light_cyan()

```elixir
@spec light_cyan() :: String.t()
```

Sets foreground color to light cyan.


### light_cyan_background()

```elixir
@spec light_cyan_background() :: String.t()
```

Sets background color to light cyan.


### light_green()

```elixir
@spec light_green() :: String.t()
```

Sets foreground color to light green.


### light_green_background()

```elixir
@spec light_green_background() :: String.t()
```

Sets background color to light green.


### light_magenta()

```elixir
@spec light_magenta() :: String.t()
```

Sets foreground color to light magenta.


### light_magenta_background()

```elixir
@spec light_magenta_background() :: String.t()
```

Sets background color to light magenta.


### light_red()

```elixir
@spec light_red() :: String.t()
```

Sets foreground color to light red.


### light_red_background()

```elixir
@spec light_red_background() :: String.t()
```

Sets background color to light red.


### light_white()

```elixir
@spec light_white() :: String.t()
```

Sets foreground color to light white.


### light_white_background()

```elixir
@spec light_white_background() :: String.t()
```

Sets background color to light white.


### light_yellow()

```elixir
@spec light_yellow() :: String.t()
```

Sets foreground color to light yellow.


### light_yellow_background()

```elixir
@spec light_yellow_background() :: String.t()
```

Sets background color to light yellow.


### magenta()

```elixir
@spec magenta() :: String.t()
```

Sets foreground color to magenta.


### magenta_background()

```elixir
@spec magenta_background() :: String.t()
```

Sets background color to magenta.


### no_underline()

```elixir
@spec no_underline() :: String.t()
```

Underline: none.


### normal()

```elixir
@spec normal() :: String.t()
```

Normal color or intensity.


### not_framed_encircled()

```elixir
@spec not_framed_encircled() :: String.t()
```

Not framed or encircled.


### not_italic()

```elixir
@spec not_italic() :: String.t()
```

Not italic.


### not_overlined()

```elixir
@spec not_overlined() :: String.t()
```

Not overlined.


### overlined()

```elixir
@spec overlined() :: String.t()
```

Overlined.


### primary_font()

```elixir
@spec primary_font() :: String.t()
```

Sets primary (default) font.


### red()

```elixir
@spec red() :: String.t()
```

Sets foreground color to red.


### red_background()

```elixir
@spec red_background() :: String.t()
```

Sets background color to red.


### reset()

```elixir
@spec reset() :: String.t()
```

Resets all attributes.


### reverse()

```elixir
@spec reverse() :: String.t()
```

Image: negative. Swap foreground and background.


### reverse_off()

```elixir
@spec reverse_off() :: String.t()
```

Image: positive. Normal foreground and background.


### syntax_colors()
*(since 1.14.0)* 
```elixir
@spec syntax_colors() :: Keyword.t(ansidata())
```

Syntax colors to be used by `Inspect`.

Those colors are used throughout Elixir's standard library,
such as `dbg/2` and `IEx`.

The colors can be changed by setting the `:ansi_syntax_colors`
in the `:elixir` application configuration. Configuration for
most built-in data types are supported: `:atom`, `:binary`,
`:boolean`, `:charlist`, `:list`, `:map`, `:nil`, `:number`,
`:string`, and `:tuple`. The default is:

    [
      atom: :cyan
      boolean: :magenta,
      charlist: :yellow,
      nil: :magenta,
      number: :yellow,
      string: :green
    ]


### underline()

```elixir
@spec underline() :: String.t()
```

Underline: single.


### white()

```elixir
@spec white() :: String.t()
```

Sets foreground color to white.


### white_background()

```elixir
@spec white_background() :: String.t()
```

Sets background color to white.


### yellow()

```elixir
@spec yellow() :: String.t()
```

Sets foreground color to yellow.


### yellow_background()

```elixir
@spec yellow_background() :: String.t()
```

Sets background color to yellow.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
