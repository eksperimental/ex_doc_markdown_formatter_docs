# StringIO 
(Elixir v1.18.0-dev)

Controls an IO device process that wraps a string.

A `StringIO` IO device can be passed as a "device" to
most of the functions in the `IO` module.

## Examples

    iex> {:ok, pid} = StringIO.open("foo")
    iex> IO.read(pid, 2)
    "fo"


## Functions

### close(pid)

```elixir
@spec close(pid()) :: {:ok, {binary(), binary()}}
```

Stops the IO device and returns the remaining input/output
buffers.

#### Examples

    iex> {:ok, pid} = StringIO.open("in")
    iex> IO.write(pid, "out")
    iex> StringIO.close(pid)
    {:ok, {"in", "out"}}


### contents(pid)

```elixir
@spec contents(pid()) :: {binary(), binary()}
```

Returns the current input/output buffers for the given IO
device.

#### Examples

    iex> {:ok, pid} = StringIO.open("in")
    iex> IO.write(pid, "out")
    iex> StringIO.contents(pid)
    {"in", "out"}


### flush(pid)

```elixir
@spec flush(pid()) :: binary()
```

Flushes the output buffer and returns its current contents.

#### Examples

    iex> {:ok, pid} = StringIO.open("in")
    iex> IO.write(pid, "out")
    iex> StringIO.flush(pid)
    "out"
    iex> StringIO.contents(pid)
    {"in", ""}


### open(string, options_or_function \\ [])

```elixir
@spec open(
  binary(),
  keyword()
) :: {:ok, pid()}
@spec open(binary(), (pid() -&gt; res)) :: {:ok, res} when res: var
```

Creates an IO device.

`string` will be the initial input of the newly created
device.

`options_or_function` can be a keyword list of options or
a function.

If options are provided, the result will be `{:ok, pid}`, returning the
IO device created. The option `:capture_prompt`, when set to `true`, causes
prompts (which are specified as arguments to `IO.get*` functions) to be
included in the device's output.

If a function is provided, the device will be created and sent to the
function. When the function returns, the device will be closed. The final
result will be a tuple with `:ok` and the result of the function.

#### Examples

    iex> {:ok, pid} = StringIO.open("foo")
    iex> IO.gets(pid, ">")
    "foo"
    iex> StringIO.contents(pid)
    {"", ""}
    
    iex> {:ok, pid} = StringIO.open("foo", capture_prompt: true)
    iex> IO.gets(pid, ">")
    "foo"
    iex> StringIO.contents(pid)
    {"", ">"}
    
    iex> StringIO.open("foo", fn pid ->
    ...>   input = IO.gets(pid, ">")
    ...>   IO.write(pid, "The input was #{input}")
    ...>   StringIO.contents(pid)
    ...> end)
    {:ok, {"", "The input was foo"}}


### open(string, options, function)
*(since 1.7.0)* 
```elixir
@spec open(binary(), keyword(), (pid() -&gt; res)) :: {:ok, res} when res: var
```

Creates an IO device.

`string` will be the initial input of the newly created
device.

The device will be created and sent to the function given.
When the function returns, the device will be closed. The final
result will be a tuple with `:ok` and the result of the function.

#### Options

- `:capture_prompt` - if set to `true`, prompts (specified as
  arguments to `IO.get*` functions) are captured in the output.
  Defaults to `false`.

- `:encoding` (since v1.10.0) - encoding of the IO device. Allowed
  values are `:unicode` (default) and `:latin1`.

#### Examples

    iex> StringIO.open("foo", [], fn pid ->
    ...>   input = IO.gets(pid, ">")
    ...>   IO.write(pid, "The input was #{input}")
    ...>   StringIO.contents(pid)
    ...> end)
    {:ok, {"", "The input was foo"}}
    
    iex> StringIO.open("foo", [capture_prompt: true], fn pid ->
    ...>   input = IO.gets(pid, ">")
    ...>   IO.write(pid, "The input was #{input}")
    ...>   StringIO.contents(pid)
    ...> end)
    {:ok, {"", ">The input was foo"}}




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
