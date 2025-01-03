# Stream 
(Elixir v1.18.0-dev)

Functions for creating and composing streams.

Streams are composable, lazy enumerables (for an introduction on
enumerables, see the `Enum` module). Any enumerable that generates
elements one by one during enumeration is called a stream. For example,
Elixir's `Range` is a stream:

    iex> range = 1..5
    1..5
    iex> Enum.map(range, &(&1 * 2))
    [2, 4, 6, 8, 10]

In the example above, as we mapped over the range, the elements being
enumerated were created one by one, during enumeration. The `Stream`
module allows us to map the range, without triggering its enumeration:

    iex> range = 1..3
    iex> stream = Stream.map(range, &(&1 * 2))
    iex> Enum.map(stream, &(&1 + 1))
    [3, 5, 7]

Note that we started with a range and then we created a stream that is
meant to multiply each element in the range by 2. At this point, no
computation was done. Only when `Enum.map/2` is called we actually
enumerate over each element in the range, multiplying it by 2 and adding 1.
We say the functions in `Stream` are *lazy* and the functions in `Enum`
are *eager*.

Due to their laziness, streams are useful when working with large
(or even infinite) collections. When chaining many operations with `Enum`,
intermediate lists are created, while `Stream` creates a recipe of
computations that are executed at a later moment. Then when the
stream is consumed later on, most commonly by using a function in
the `Enum` module, the stream will emit its elements one by one.

Let's see another example:

    1..3
    |> Enum.map(&IO.inspect(&1))
    |> Enum.map(&(&1 * 2))
    |> Enum.map(&IO.inspect(&1))
    1
    2
    3
    2
    4
    6
    #=> [2, 4, 6]

Note that we first printed each element in the list, then multiplied each
element by 2 and finally printed each new value. In this example, the list
was enumerated three times. Let's see an example with streams:

    stream = 1..3
    |> Stream.map(&IO.inspect(&1))
    |> Stream.map(&(&1 * 2))
    |> Stream.map(&IO.inspect(&1))
    Enum.to_list(stream)
    1
    2
    2
    4
    3
    6
    #=> [2, 4, 6]

Although the end result is the same, the order in which the elements were
printed changed\! With streams, we print the first element and then print
its double. In this example, the list was enumerated just once\!

That's what we meant when we said earlier that streams are composable,
lazy enumerables. Note that we could call `Stream.map/2` multiple times,
effectively composing the streams and keeping them lazy. The computations
are only performed when you call a function from the `Enum` module.

Like with `Enum`, the functions in this module work in linear time. This
means that, the time it takes to perform an operation grows at the same
rate as the length of the list. This is expected on operations such as
`Stream.map/2`. After all, if we want to traverse every element on a
stream, the longer the stream, the more elements we need to traverse,
and the longer it will take.

## Creating Streams

There are many functions in Elixir's standard library that return
streams, some examples are:

- `IO.stream/2`         - streams input lines, one by one
- `URI.query_decoder/1` - decodes a query string, pair by pair

This module also provides many convenience functions for creating streams,
like `Stream.cycle/1`, `Stream.unfold/2`, `Stream.resource/3` and more.

> #### Do not check for `Stream` structs
> 
> While some functions in this module may return the `Stream` struct,
> you must never explicitly check for the `Stream` struct, as streams
> may come in several shapes, such as `IO.Stream`, `File.Stream`, or
> even `Range`s.
> 
> The functions in this module only guarantee to return enumerables
> and their implementation (structs, anonymous functions, etc) may
> change at any time. For example, a function that returns an anonymous
> function today may return a struct in future releases.
> 
> Instead of checking for a particular type, you must instead write
> assertive code that assumes you have an enumerable, using the functions
> in the `Enum` or `Stream` module accordingly.

## Types

### acc()

```elixir
@type acc() :: any()
```



### default()

```elixir
@type default() :: any()
```



### element()

```elixir
@type element() :: any()
```



### index()

```elixir
@type index() :: non_neg_integer()
```

Zero-based index.

### timer()

```elixir
@type timer() :: non_neg_integer() | :infinity
```



## Functions

### chunk_by(enum, fun)

```elixir
@spec chunk_by(Enumerable.t(), (element() -&gt; any())) :: Enumerable.t()
```

Chunks the `enum` by buffering elements for which `fun` returns the same value.

Elements are only emitted when `fun` returns a new value or the `enum` finishes.

#### Examples

    iex> stream = Stream.chunk_by([1, 2, 2, 3, 4, 4, 6, 7, 7], &(rem(&1, 2) == 1))
    iex> Enum.to_list(stream)
    [[1], [2, 2], [3], [4, 4, 6], [7, 7]]

### chunk_every(enum, count)
*(since 1.5.0)* 
```elixir
@spec chunk_every(Enumerable.t(), pos_integer()) :: Enumerable.t()
```

Shortcut to `chunk_every(enum, count, count)`.

### chunk_every(enum, count, step, leftover \\ [])
*(since 1.5.0)* 
```elixir
@spec chunk_every(
  Enumerable.t(),
  pos_integer(),
  pos_integer(),
  Enumerable.t() | :discard
) ::
  Enumerable.t()
```

Streams the enumerable in chunks, containing `count` elements each,
where each new chunk starts `step` elements into the enumerable.

`step` is optional and, if not passed, defaults to `count`, i.e.
chunks do not overlap. Chunking will stop as soon as the collection
ends or when we emit an incomplete chunk.

If the last chunk does not have `count` elements to fill the chunk,
elements are taken from `leftover` to fill in the chunk. If `leftover`
does not have enough elements to fill the chunk, then a partial chunk
is returned with less than `count` elements.

If `:discard` is given in `leftover`, the last chunk is discarded
unless it has exactly `count` elements.

#### Examples

    iex> Stream.chunk_every([1, 2, 3, 4, 5, 6], 2) |> Enum.to_list()
    [[1, 2], [3, 4], [5, 6]]
    
    iex> Stream.chunk_every([1, 2, 3, 4, 5, 6], 3, 2, :discard) |> Enum.to_list()
    [[1, 2, 3], [3, 4, 5]]
    
    iex> Stream.chunk_every([1, 2, 3, 4, 5, 6], 3, 2, [7]) |> Enum.to_list()
    [[1, 2, 3], [3, 4, 5], [5, 6, 7]]
    
    iex> Stream.chunk_every([1, 2, 3, 4, 5, 6], 3, 3, []) |> Enum.to_list()
    [[1, 2, 3], [4, 5, 6]]
    
    iex> Stream.chunk_every([1, 2, 3, 4], 3, 3, Stream.cycle([0])) |> Enum.to_list()
    [[1, 2, 3], [4, 0, 0]]

### chunk_while(enum, acc, chunk_fun, after_fun)
*(since 1.5.0)* 
```elixir
@spec chunk_while(
  Enumerable.t(),
  acc(),
  (element(), acc() -&gt; {:cont, chunk, acc()} | {:cont, acc()} | {:halt, acc()}),
  (acc() -&gt; {:cont, chunk, acc()} | {:cont, acc()})
) :: Enumerable.t()
when chunk: any()
```

Chunks the `enum` with fine grained control when every chunk is emitted.

`chunk_fun` receives the current element and the accumulator and
must return `{:cont, element, acc}` to emit the given chunk and
continue with accumulator or `{:cont, acc}` to not emit any chunk
and continue with the return accumulator.

`after_fun` is invoked when iteration is done and must also return
`{:cont, element, acc}` or `{:cont, acc}`.

#### Examples

    iex> chunk_fun = fn element, acc ->
    ...>   if rem(element, 2) == 0 do
    ...>     {:cont, Enum.reverse([element | acc]), []}
    ...>   else
    ...>     {:cont, [element | acc]}
    ...>   end
    ...> end
    iex> after_fun = fn
    ...>   [] -> {:cont, []}
    ...>   acc -> {:cont, Enum.reverse(acc), []}
    ...> end
    iex> stream = Stream.chunk_while(1..10, [], chunk_fun, after_fun)
    iex> Enum.to_list(stream)
    [[1, 2], [3, 4], [5, 6], [7, 8], [9, 10]]

### concat(enumerables)

```elixir
@spec concat(Enumerable.t()) :: Enumerable.t()
```

Creates a stream that enumerates each enumerable in an enumerable.

#### Examples

    iex> stream = Stream.concat([1..3, 4..6, 7..9])
    iex> Enum.to_list(stream)
    [1, 2, 3, 4, 5, 6, 7, 8, 9]

### concat(first, second)

```elixir
@spec concat(Enumerable.t(), Enumerable.t()) :: Enumerable.t()
```

Creates a stream that enumerates the first argument, followed by the second.

#### Examples

    iex> stream = Stream.concat(1..3, 4..6)
    iex> Enum.to_list(stream)
    [1, 2, 3, 4, 5, 6]
    
    iex> stream1 = Stream.cycle([1, 2, 3])
    iex> stream2 = Stream.cycle([4, 5, 6])
    iex> stream = Stream.concat(stream1, stream2)
    iex> Enum.take(stream, 6)
    [1, 2, 3, 1, 2, 3]

### cycle(enumerable)

```elixir
@spec cycle(Enumerable.t()) :: Enumerable.t()
```

Creates a stream that cycles through the given enumerable,
infinitely.

#### Examples

    iex> stream = Stream.cycle([1, 2, 3])
    iex> Enum.take(stream, 5)
    [1, 2, 3, 1, 2]

### dedup(enum)

```elixir
@spec dedup(Enumerable.t()) :: Enumerable.t()
```

Creates a stream that only emits elements if they are different from the last emitted element.

This function only ever needs to store the last emitted element.

Elements are compared using `===/2`.

#### Examples

    iex> Stream.dedup([1, 2, 3, 3, 2, 1]) |> Enum.to_list()
    [1, 2, 3, 2, 1]

### dedup_by(enum, fun)

```elixir
@spec dedup_by(Enumerable.t(), (element() -&gt; term())) :: Enumerable.t()
```

Creates a stream that only emits elements if the result of calling `fun` on the element is
different from the (stored) result of calling `fun` on the last emitted element.

#### Examples

    iex> Stream.dedup_by([{1, :x}, {2, :y}, {2, :z}, {1, :x}], fn {x, _} -> x end) |> Enum.to_list()
    [{1, :x}, {2, :y}, {1, :x}]

### drop(enum, n)

```elixir
@spec drop(Enumerable.t(), integer()) :: Enumerable.t()
```

Lazily drops the next `n` elements from the enumerable.

If a negative `n` is given, it will drop the last `n` elements from
the collection. Note that the mechanism by which this is implemented
will delay the emission of any element until `n` additional elements have
been emitted by the enum.

#### Examples

    iex> stream = Stream.drop(1..10, 5)
    iex> Enum.to_list(stream)
    [6, 7, 8, 9, 10]
    
    iex> stream = Stream.drop(1..10, -5)
    iex> Enum.to_list(stream)
    [1, 2, 3, 4, 5]

### drop_every(enum, nth)

```elixir
@spec drop_every(Enumerable.t(), non_neg_integer()) :: Enumerable.t()
```

Creates a stream that drops every `nth` element from the enumerable.

The first element is always dropped, unless `nth` is 0.

`nth` must be a non-negative integer.

#### Examples

    iex> stream = Stream.drop_every(1..10, 2)
    iex> Enum.to_list(stream)
    [2, 4, 6, 8, 10]
    
    iex> stream = Stream.drop_every(1..1000, 1)
    iex> Enum.to_list(stream)
    []
    
    iex> stream = Stream.drop_every([1, 2, 3, 4, 5], 0)
    iex> Enum.to_list(stream)
    [1, 2, 3, 4, 5]

### drop_while(enum, fun)

```elixir
@spec drop_while(Enumerable.t(), (element() -&gt; as_boolean(term()))) :: Enumerable.t()
```

Lazily drops elements of the enumerable while the given
function returns a truthy value.

#### Examples

    iex> stream = Stream.drop_while(1..10, &(&1 <= 5))
    iex> Enum.to_list(stream)
    [6, 7, 8, 9, 10]

### duplicate(value, n)
*(since 1.14.0)* 
```elixir
@spec duplicate(any(), non_neg_integer()) :: Enumerable.t()
```

Duplicates the given element `n` times in a stream.

`n` is an integer greater than or equal to `0`.

If `n` is `0`, an empty stream is returned.

#### Examples

    iex> stream = Stream.duplicate("hello", 0)
    iex> Enum.to_list(stream)
    []
    
    iex> stream = Stream.duplicate("hi", 1)
    iex> Enum.to_list(stream)
    ["hi"]
    
    iex> stream = Stream.duplicate("bye", 2)
    iex> Enum.to_list(stream)
    ["bye", "bye"]
    
    iex> stream = Stream.duplicate([1, 2], 3)
    iex> Enum.to_list(stream)
    [[1, 2], [1, 2], [1, 2]]

### each(enum, fun)

```elixir
@spec each(Enumerable.t(), (element() -&gt; term())) :: Enumerable.t()
```

Executes the given function for each element.

The values in the stream do not change, therefore this
function is useful for adding side effects (like printing)
to a stream. See `map/2` if producing a different stream
is desired.

#### Examples

    iex> stream = Stream.each([1, 2, 3], fn x -> send(self(), x) end)
    iex> Enum.to_list(stream)
    iex> receive do: (x when is_integer(x) -> x)
    1
    iex> receive do: (x when is_integer(x) -> x)
    2
    iex> receive do: (x when is_integer(x) -> x)
    3

### filter(enum, fun)

```elixir
@spec filter(Enumerable.t(), (element() -&gt; as_boolean(term()))) :: Enumerable.t()
```

Creates a stream that filters elements according to
the given function on enumeration.

#### Examples

    iex> stream = Stream.filter([1, 2, 3], fn x -> rem(x, 2) == 0 end)
    iex> Enum.to_list(stream)
    [2]

### flat_map(enum, mapper)

```elixir
@spec flat_map(Enumerable.t(), (element() -&gt; Enumerable.t())) :: Enumerable.t()
```

Maps the given `fun` over `enumerable` and flattens the result.

This function returns a new stream built by appending the result of invoking `fun`
on each element of `enumerable` together.

#### Examples

    iex> stream = Stream.flat_map([1, 2, 3], fn x -> [x, x * 2] end)
    iex> Enum.to_list(stream)
    [1, 2, 2, 4, 3, 6]
    
    iex> stream = Stream.flat_map([1, 2, 3], fn x -> [[x]] end)
    iex> Enum.to_list(stream)
    [[1], [2], [3]]

### from_index(fun_or_offset \\ 0)
*(since 1.17.0)* 
```elixir
@spec from_index(integer()) :: Enumerable.t(integer())
@spec from_index((integer() -&gt; return_value)) :: Enumerable.t(return_value)
when return_value: term()
```

Builds a stream from an index, either starting from offset, or given by function.

May receive a function or an integer offset.

If an `offset` is given, it will emit elements from offset.

If a `function` is given, it will invoke the function with
elements from offset.

#### Examples

    iex> Stream.from_index() |> Enum.take(3)
    [0, 1, 2]
    
    iex> Stream.from_index(1) |> Enum.take(3)
    [1, 2, 3]
    
    iex> Stream.from_index(fn x -> x * 10 end) |> Enum.take(3)
    [0, 10, 20]

### intersperse(enumerable, intersperse_element)
*(since 1.6.0)* 
```elixir
@spec intersperse(Enumerable.t(), any()) :: Enumerable.t()
```

Lazily intersperses `intersperse_element` between each element of the enumeration.

#### Examples

    iex> Stream.intersperse([1, 2, 3], 0) |> Enum.to_list()
    [1, 0, 2, 0, 3]
    
    iex> Stream.intersperse([1], 0) |> Enum.to_list()
    [1]
    
    iex> Stream.intersperse([], 0) |> Enum.to_list()
    []

### interval(n)

```elixir
@spec interval(timer()) :: Enumerable.t()
```

Creates a stream that emits a value after the given period `n`
in milliseconds.

The values emitted are an increasing counter starting at `0`.
This operation will block the caller by the given interval
every time a new element is streamed.

Do not use this function to generate a sequence of numbers.
If blocking the caller process is not necessary, use
`Stream.iterate(0, & &1 + 1)` instead.

#### Examples

    iex> Stream.interval(10) |> Enum.take(10)
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

### into(enum, collectable, transform \\ fn x -&gt; x end)

```elixir
@spec into(Enumerable.t(), Collectable.t(), (term() -&gt; term())) :: Enumerable.t()
```

Injects the stream values into the given collectable as a side-effect.

This function is often used with `run/1` since any evaluation
is delayed until the stream is executed. See `run/1` for an example.

### iterate(start_value, next_fun)

```elixir
@spec iterate(element(), (element() -&gt; element())) :: Enumerable.t()
```

Emits a sequence of values, starting with `start_value`.

Successive values are generated by calling `next_fun`
on the previous value.

#### Examples

    iex> Stream.iterate(1, &(&1 * 2)) |> Enum.take(5)
    [1, 2, 4, 8, 16]

### map(enum, fun)

```elixir
@spec map(Enumerable.t(), (element() -&gt; any())) :: Enumerable.t()
```

Creates a stream that will apply the given function on
enumeration.

#### Examples

    iex> stream = Stream.map([1, 2, 3], fn x -> x * 2 end)
    iex> Enum.to_list(stream)
    [2, 4, 6]

### map_every(enum, nth, fun)
*(since 1.4.0)* 
```elixir
@spec map_every(Enumerable.t(), non_neg_integer(), (element() -&gt; any())) ::
  Enumerable.t()
```

Creates a stream that will apply the given function on
every `nth` element from the enumerable.

The first element is always passed to the given function.

`nth` must be a non-negative integer.

#### Examples

    iex> stream = Stream.map_every(1..10, 2, fn x -> x * 2 end)
    iex> Enum.to_list(stream)
    [2, 2, 6, 4, 10, 6, 14, 8, 18, 10]
    
    iex> stream = Stream.map_every([1, 2, 3, 4, 5], 1, fn x -> x * 2 end)
    iex> Enum.to_list(stream)
    [2, 4, 6, 8, 10]
    
    iex> stream = Stream.map_every(1..5, 0, fn x -> x * 2 end)
    iex> Enum.to_list(stream)
    [1, 2, 3, 4, 5]

### reject(enum, fun)

```elixir
@spec reject(Enumerable.t(), (element() -&gt; as_boolean(term()))) :: Enumerable.t()
```

Creates a stream that will reject elements according to
the given function on enumeration.

#### Examples

    iex> stream = Stream.reject([1, 2, 3], fn x -> rem(x, 2) == 0 end)
    iex> Enum.to_list(stream)
    [1, 3]

### repeatedly(generator_fun)

```elixir
@spec repeatedly((-&gt; element())) :: Enumerable.t()
```

Returns a stream generated by calling `generator_fun` repeatedly.

#### Examples

    # Although not necessary, let's seed the random algorithm
    iex> :rand.seed(:exsss, {1, 2, 3})
    iex> Stream.repeatedly(&:rand.uniform/0) |> Enum.take(3)
    [0.5455598952593053, 0.6039309974353404, 0.6684893034823949]

### resource(start_fun, next_fun, after_fun)

```elixir
@spec resource((-&gt; acc()), (acc() -&gt; {[element()], acc()} | {:halt, acc()}), (acc() -&gt;
                                                                          term())) ::
  Enumerable.t()
```

Emits a sequence of values for the given resource.

Similar to `transform/3` but the initial accumulated value is
computed lazily via `start_fun` and executes an `after_fun` at
the end of enumeration (both in cases of success and failure).

Successive values are generated by calling `next_fun` with the
previous accumulator (the initial value being the result returned
by `start_fun`) and it must return a tuple containing a list
of elements to be emitted and the next accumulator. The enumeration
finishes if it returns `{:halt, acc}`.

As the function name suggests, this function is useful to stream values from
resources.

#### Examples

    Stream.resource(
      fn -> File.open!("sample") end,
      fn file ->
        case IO.read(file, :line) do
          data when is_binary(data) -> {[data], file}
          _ -> {:halt, file}
        end
      end,
      fn file -> File.close(file) end
    )
    
    iex> Stream.resource(
    ...>  fn ->
    ...>    {:ok, pid} = StringIO.open("string")
    ...>    pid
    ...>  end,
    ...>  fn pid ->
    ...>    case IO.getn(pid, "", 1) do
    ...>      :eof -> {:halt, pid}
    ...>      char -> {[char], pid}
    ...>    end
    ...>  end,
    ...>  fn pid -> StringIO.close(pid) end
    ...> ) |> Enum.to_list()
    ["s", "t", "r", "i", "n", "g"]

### run(stream)

```elixir
@spec run(Enumerable.t()) :: :ok
```

Runs the given stream.

This is useful when a stream needs to be run, for side effects,
and there is no interest in its return result.

#### Examples

Open up a file, replace all `#` by `%` and stream to another file
without loading the whole file in memory:

    File.stream!("/path/to/file")
    |> Stream.map(&String.replace(&1, "#", "%"))
    |> Stream.into(File.stream!("/path/to/other/file"))
    |> Stream.run()

No computation will be done until we call one of the `Enum` functions
or `run/1`.

### scan(enum, fun)

```elixir
@spec scan(Enumerable.t(), (element(), acc() -&gt; any())) :: Enumerable.t()
```

Creates a stream that applies the given function to each
element, emits the result and uses the same result as the accumulator
for the next computation. Uses the first element in the enumerable
as the starting value.

#### Examples

    iex> stream = Stream.scan(1..5, &(&1 + &2))
    iex> Enum.to_list(stream)
    [1, 3, 6, 10, 15]

### scan(enum, acc, fun)

```elixir
@spec scan(Enumerable.t(), acc(), (element(), acc() -&gt; any())) :: Enumerable.t()
```

Creates a stream that applies the given function to each
element, emits the result and uses the same result as the accumulator
for the next computation. Uses the given `acc` as the starting value.

#### Examples

    iex> stream = Stream.scan(1..5, 0, &(&1 + &2))
    iex> Enum.to_list(stream)
    [1, 3, 6, 10, 15]

### take(enum, count)

```elixir
@spec take(Enumerable.t(), integer()) :: Enumerable.t()
```

Lazily takes the next `count` elements from the enumerable and stops
enumeration.

If a negative `count` is given, the last `count` values will be taken.
For such, the collection is fully enumerated keeping up to `2 * count`
elements in memory. Once the end of the collection is reached,
the last `count` elements will be executed. Therefore, using
a negative `count` on an infinite collection will never return.

#### Examples

    iex> stream = Stream.take(1..100, 5)
    iex> Enum.to_list(stream)
    [1, 2, 3, 4, 5]
    
    iex> stream = Stream.take(1..100, -5)
    iex> Enum.to_list(stream)
    [96, 97, 98, 99, 100]
    
    iex> stream = Stream.cycle([1, 2, 3]) |> Stream.take(5)
    iex> Enum.to_list(stream)
    [1, 2, 3, 1, 2]

### take_every(enum, nth)

```elixir
@spec take_every(Enumerable.t(), non_neg_integer()) :: Enumerable.t()
```

Creates a stream that takes every `nth` element from the enumerable.

The first element is always included, unless `nth` is 0.

`nth` must be a non-negative integer.

#### Examples

    iex> stream = Stream.take_every(1..10, 2)
    iex> Enum.to_list(stream)
    [1, 3, 5, 7, 9]
    
    iex> stream = Stream.take_every([1, 2, 3, 4, 5], 1)
    iex> Enum.to_list(stream)
    [1, 2, 3, 4, 5]
    
    iex> stream = Stream.take_every(1..1000, 0)
    iex> Enum.to_list(stream)
    []

### take_while(enum, fun)

```elixir
@spec take_while(Enumerable.t(), (element() -&gt; as_boolean(term()))) :: Enumerable.t()
```

Lazily takes elements of the enumerable while the given
function returns a truthy value.

#### Examples

    iex> stream = Stream.take_while(1..100, &(&1 <= 5))
    iex> Enum.to_list(stream)
    [1, 2, 3, 4, 5]

### timer(n)

```elixir
@spec timer(timer()) :: Enumerable.t()
```

Creates a stream that emits a single value after `n` milliseconds.

The value emitted is `0`. This operation will block the caller by
the given time until the element is streamed.

#### Examples

    iex> Stream.timer(10) |> Enum.to_list()
    [0]

### transform(enum, acc, reducer)

```elixir
@spec transform(Enumerable.t(), acc, fun) :: Enumerable.t()
when fun: (element(), acc -&gt; {Enumerable.t(), acc} | {:halt, acc}), acc: term()
```

Transforms an existing stream.

It expects an accumulator and a function that receives two arguments,
the stream element and the updated accumulator. It must return a tuple,
where the first element is a new stream (often a list) or the atom `:halt`,
and the second element is the accumulator to be used by the next element.

Note: this function is equivalent to `Enum.flat_map_reduce/3`, except this
function does not return the accumulator once the stream is processed.

#### Examples

`Stream.transform/3` is useful as it can be used as the basis to implement
many of the functions defined in this module. For example, we can implement
`Stream.take(enum, n)` as follows:

    iex> enum = 1001..9999
    iex> n = 3
    iex> stream = Stream.transform(enum, 0, fn i, acc ->
    ...>   if acc < n, do: {[i], acc + 1}, else: {:halt, acc}
    ...> end)
    iex> Enum.to_list(stream)
    [1001, 1002, 1003]

`Stream.transform/5` further generalizes this function to allow wrapping
around resources.

### transform(enum, start_fun, reducer, after_fun)

```elixir
@spec transform(Enumerable.t(), start_fun, reducer, after_fun) :: Enumerable.t()
when start_fun: (-&gt; acc),
     reducer: (element(), acc -&gt; {Enumerable.t(), acc} | {:halt, acc}),
     after_fun: (acc -&gt; term()),
     acc: term()
```

Similar to `Stream.transform/5`, except `last_fun` is not supplied.

This function can be seen as a combination of `Stream.resource/3` with
`Stream.transform/3`.

### transform(enum, start_fun, reducer, last_fun, after_fun)

```elixir
@spec transform(Enumerable.t(), start_fun, reducer, last_fun, after_fun) ::
  Enumerable.t()
when start_fun: (-&gt; acc),
     reducer: (element(), acc -&gt; {Enumerable.t(), acc} | {:halt, acc}),
     last_fun: (acc -&gt; {Enumerable.t(), acc} | {:halt, acc}),
     after_fun: (acc -&gt; term()),
     acc: term()
```

Transforms an existing stream with function-based start, last, and after
callbacks.

Once transformation starts, `start_fun` is invoked to compute the initial
accumulator. Then, for each element in the enumerable, the `reducer` function
is invoked with the element and the accumulator, returning new elements and a
new accumulator, as in `transform/3`.

Once the collection is done, `last_fun` is invoked with the accumulator to
emit any remaining items. Then `after_fun` is invoked, to close any resource,
but not emitting any new items. `last_fun` is only invoked if the given
enumerable terminates successfully (either because it is done or it halted
itself). `after_fun` is always invoked, therefore `after_fun` must be the
one used for closing resources.

### unfold(next_acc, next_fun)

```elixir
@spec unfold(acc(), (acc() -&gt; {element(), acc()} | nil)) :: Enumerable.t()
```

Emits a sequence of values for the given accumulator.

Successive values are generated by calling `next_fun` with the previous
accumulator and it must return a tuple with the current value and next
accumulator. The enumeration finishes if it returns `nil`.

#### Examples

To create a stream that counts down and stops before zero:

    iex> Stream.unfold(5, fn
    ...>   0 -> nil
    ...>   n -> {n, n - 1}
    ...> end) |> Enum.to_list()
    [5, 4, 3, 2, 1]

If `next_fun` never returns `nil`, the returned stream is *infinite*:

    iex> Stream.unfold(0, fn
    ...>   n -> {n, n + 1}
    ...> end) |> Enum.take(10)
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    
    iex> Stream.unfold(1, fn
    ...>   n -> {n, n * 2}
    ...> end) |> Enum.take(10)
    [1, 2, 4, 8, 16, 32, 64, 128, 256, 512]

### uniq(enum)

```elixir
@spec uniq(Enumerable.t()) :: Enumerable.t()
```

Creates a stream that only emits elements if they are unique.

Keep in mind that, in order to know if an element is unique
or not, this function needs to store all unique values emitted
by the stream. Therefore, if the stream is infinite, the number
of elements stored will grow infinitely, never being garbage-collected.

#### Examples

    iex> Stream.uniq([1, 2, 3, 3, 2, 1]) |> Enum.to_list()
    [1, 2, 3]

### uniq_by(enum, fun)

```elixir
@spec uniq_by(Enumerable.t(), (element() -&gt; term())) :: Enumerable.t()
```

Creates a stream that only emits elements if they are unique, by removing the
elements for which function `fun` returned duplicate elements.

The function `fun` maps every element to a term which is used to
determine if two elements are duplicates.

Keep in mind that, in order to know if an element is unique
or not, this function needs to store all unique values emitted
by the stream. Therefore, if the stream is infinite, the number
of elements stored will grow infinitely, never being garbage-collected.

#### Example

    iex> Stream.uniq_by([{1, :x}, {2, :y}, {1, :z}], fn {x, _} -> x end) |> Enum.to_list()
    [{1, :x}, {2, :y}]
    
    iex> Stream.uniq_by([a: {:tea, 2}, b: {:tea, 2}, c: {:coffee, 1}], fn {_, y} -> y end) |> Enum.to_list()
    [a: {:tea, 2}, c: {:coffee, 1}]

### with_index(enum, fun_or_offset \\ 0)

```elixir
@spec with_index(Enumerable.t(), integer()) :: Enumerable.t({element(), integer()})
@spec with_index(Enumerable.t(), (element(), index() -&gt; return_value)) ::
  Enumerable.t(return_value)
when return_value: term()
```

Creates a stream where each element in the enumerable will
be wrapped in a tuple alongside its index or according to a given function.

May receive a function or an integer offset.

If an `offset` is given, it will index from the given offset instead of from
zero.

If a `function` is given, it will index by invoking the function for each
element and index (zero-based) of the enumerable.

#### Examples

    iex> stream = Stream.with_index([1, 2, 3])
    iex> Enum.to_list(stream)
    [{1, 0}, {2, 1}, {3, 2}]
    
    iex> stream = Stream.with_index([1, 2, 3], 3)
    iex> Enum.to_list(stream)
    [{1, 3}, {2, 4}, {3, 5}]
    
    iex> stream = Stream.with_index([1, 2, 3], fn x, index -> x + index end)
    iex> Enum.to_list(stream)
    [1, 3, 5]

### zip(enumerables)
*(since 1.4.0)* 
```elixir
@spec zip(enumerables) :: Enumerable.t()
when enumerables: [Enumerable.t()] | Enumerable.t()
```

Zips corresponding elements from a finite collection of enumerables
into one stream of tuples.

The zipping finishes as soon as any enumerable in the given collection completes.

#### Examples

    iex> concat = Stream.concat(1..3, 4..6)
    iex> cycle = Stream.cycle(["foo", "bar", "baz"])
    iex> Stream.zip([concat, [:a, :b, :c], cycle]) |> Enum.to_list()
    [{1, :a, "foo"}, {2, :b, "bar"}, {3, :c, "baz"}]

### zip(enumerable1, enumerable2)

```elixir
@spec zip(Enumerable.t(), Enumerable.t()) :: Enumerable.t()
```

Zips two enumerables together, lazily.

Because a list of two-element tuples with atoms as the first
tuple element is a keyword list (`Keyword`), zipping a first `Stream`
of atoms with a second `Stream` of any kind creates a `Stream`
that generates a keyword list.

The zipping finishes as soon as either enumerable completes.

#### Examples

    iex> concat = Stream.concat(1..3, 4..6)
    iex> cycle = Stream.cycle([:a, :b, :c])
    iex> Stream.zip(concat, cycle) |> Enum.to_list()
    [{1, :a}, {2, :b}, {3, :c}, {4, :a}, {5, :b}, {6, :c}]
    iex> Stream.zip(cycle, concat) |> Enum.to_list()
    [a: 1, b: 2, c: 3, a: 4, b: 5, c: 6]

### zip_with(enumerables, zip_fun)
*(since 1.12.0)* 
```elixir
@spec zip_with(enumerables, (Enumerable.t() -&gt; term())) :: Enumerable.t()
when enumerables: [Enumerable.t()] | Enumerable.t()
```

Lazily zips corresponding elements from a finite collection of enumerables into a new
enumerable, transforming them with the `zip_fun` function as it goes.

The first element from each of the enums in `enumerables` will be put into a list which is then passed to
the one-arity `zip_fun` function. Then, the second elements from each of the enums are put into a list and passed to
`zip_fun`, and so on until any one of the enums in `enumerables` completes.

Returns a new enumerable with the results of calling `zip_fun`.

#### Examples

    iex> concat = Stream.concat(1..3, 4..6)
    iex> Stream.zip_with([concat, concat], fn [a, b] -> a + b end) |> Enum.to_list()
    [2, 4, 6, 8, 10, 12]
    
    iex> concat = Stream.concat(1..3, 4..6)
    iex> Stream.zip_with([concat, concat, 1..3], fn [a, b, c] -> a + b + c end) |> Enum.to_list()
    [3, 6, 9]

### zip_with(enumerable1, enumerable2, zip_fun)
*(since 1.12.0)* 
```elixir
@spec zip_with(Enumerable.t(), Enumerable.t(), (term(), term() -&gt; term())) ::
  Enumerable.t()
```

Lazily zips corresponding elements from two enumerables into a new one, transforming them with
the `zip_fun` function as it goes.

The `zip_fun` will be called with the first element from `enumerable1` and the first
element from `enumerable2`, then with the second element from each, and so on until
either one of the enumerables completes.

#### Examples

    iex> concat = Stream.concat(1..3, 4..6)
    iex> Stream.zip_with(concat, concat, fn a, b -> a + b end) |> Enum.to_list()
    [2, 4, 6, 8, 10, 12]



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
