# Enum 
(Elixir v1.18.0-dev)

Functions for working with collections (known as enumerables).

In Elixir, an enumerable is any data type that implements the
`Enumerable` protocol. `List`s (`[1, 2, 3]`), `Map`s (`%{foo: 1, bar: 2}`)
and `Range`s (`1..3`) are common data types used as enumerables:

    iex> Enum.map([1, 2, 3], fn x -> x * 2 end)
    [2, 4, 6]
    
    iex> Enum.sum([1, 2, 3])
    6
    
    iex> Enum.map(1..3, fn x -> x * 2 end)
    [2, 4, 6]
    
    iex> Enum.sum(1..3)
    6
    
    iex> map = %{"a" => 1, "b" => 2}
    iex> Enum.map(map, fn {k, v} -> {k, v * 2} end)
    [{"a", 2}, {"b", 4}]

Many other enumerables exist in the language, such as `MapSet`s
and the data type returned by `File.stream!/3` which allows a file to be
traversed as if it was an enumerable.

For a general overview of all functions in the `Enum` module, see
[the `Enum` cheatsheet](enum-cheat.cheatmd).

The functions in this module work in linear time. This means that, the
time it takes to perform an operation grows at the same rate as the length
of the enumerable. This is expected on operations such as `Enum.map/2`.
After all, if we want to traverse every element on a list, the longer the
list, the more elements we need to traverse, and the longer it will take.

This linear behavior should also be expected on operations like `count/1`,
`member?/2`, `at/2` and similar. While Elixir does allow data types to
provide performant variants for such operations, you should not expect it
to always be available, since the `Enum` module is meant to work with a
large variety of data types and not all data types can provide optimized
behavior.

Finally, note the functions in the `Enum` module are eager: they will
traverse the enumerable as soon as they are invoked. This is particularly
dangerous when working with infinite enumerables. In such cases, you should
use the `Stream` module, which allows you to lazily express computations,
without traversing collections, and work with possibly infinite collections.
See the `Stream` module for examples and documentation.

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
@type index() :: integer()
```

Zero-based index. It can also be a negative integer.

### t()

```elixir
@type t() :: Enumerable.t()
```



## Functions

### all?(enumerable)

```elixir
@spec all?(t()) :: boolean()
```

Returns `true` if all elements in `enumerable` are truthy.

When an element has a falsy value (`false` or `nil`) iteration stops immediately
and `false` is returned. In all other cases `true` is returned.

#### Examples

    iex> Enum.all?([1, 2, 3])
    true
    
    iex> Enum.all?([1, nil, 3])
    false
    
    iex> Enum.all?([])
    true

### all?(enumerable, fun)

```elixir
@spec all?(t(), (element() -&gt; as_boolean(term()))) :: boolean()
```

Returns `true` if `fun.(element)` is truthy for all elements in `enumerable`.

Iterates over `enumerable` and invokes `fun` on each element. If `fun` ever
returns a falsy value (`false` or `nil`), iteration stops immediately and
`false` is returned. Otherwise, `true` is returned.

#### Examples

    iex> Enum.all?([2, 4, 6], fn x -> rem(x, 2) == 0 end)
    true
    
    iex> Enum.all?([2, 3, 4], fn x -> rem(x, 2) == 0 end)
    false
    
    iex> Enum.all?([], fn _ -> nil end)
    true

As the last example shows, `Enum.all?/2` returns `true` if `enumerable` is
empty, regardless of `fun`. In an empty enumerable there is no element for
which `fun` returns a falsy value, so the result must be `true`. This is a
well-defined logical argument for empty collections.

### any?(enumerable)

```elixir
@spec any?(t()) :: boolean()
```

Returns `true` if at least one element in `enumerable` is truthy.

When an element has a truthy value (neither `false` nor `nil`) iteration stops
immediately and `true` is returned. In all other cases `false` is returned.

#### Examples

    iex> Enum.any?([false, false, false])
    false
    
    iex> Enum.any?([false, true, false])
    true
    
    iex> Enum.any?([])
    false

### any?(enumerable, fun)

```elixir
@spec any?(t(), (element() -&gt; as_boolean(term()))) :: boolean()
```

Returns `true` if `fun.(element)` is truthy for at least one element in `enumerable`.

Iterates over the `enumerable` and invokes `fun` on each element. When an invocation
of `fun` returns a truthy value (neither `false` nor `nil`) iteration stops
immediately and `true` is returned. In all other cases `false` is returned.

#### Examples

    iex> Enum.any?([2, 4, 6], fn x -> rem(x, 2) == 1 end)
    false
    
    iex> Enum.any?([2, 3, 4], fn x -> rem(x, 2) == 1 end)
    true
    
    iex> Enum.any?([], fn x -> x > 0 end)
    false

### at(enumerable, index, default \\ nil)

```elixir
@spec at(t(), index(), default()) :: element() | default()
```

Finds the element at the given `index` (zero-based).

Returns `default` if `index` is out of bounds.

A negative `index` can be passed, which means the `enumerable` is
enumerated once and the `index` is counted from the end (for example,
`-1` finds the last element).

#### Examples

    iex> Enum.at([2, 4, 6], 0)
    2
    
    iex> Enum.at([2, 4, 6], 2)
    6
    
    iex> Enum.at([2, 4, 6], 4)
    nil
    
    iex> Enum.at([2, 4, 6], 4, :none)
    :none

### chunk_by(enumerable, fun)

```elixir
@spec chunk_by(t(), (element() -&gt; any())) :: [list()]
```

Splits enumerable on every element for which `fun` returns a new
value.

Returns a list of lists.

#### Examples

    iex> Enum.chunk_by([1, 2, 2, 3, 4, 4, 6, 7, 7], &(rem(&1, 2) == 1))
    [[1], [2, 2], [3], [4, 4, 6], [7, 7]]

### chunk_every(enumerable, count)
*(since 1.5.0)* 
```elixir
@spec chunk_every(t(), pos_integer()) :: [list()]
```

Shortcut to `chunk_every(enumerable, count, count)`.

### chunk_every(enumerable, count, step, leftover \\ [])
*(since 1.5.0)* 
```elixir
@spec chunk_every(t(), pos_integer(), pos_integer(), t() | :discard) :: [list()]
```

Returns list of lists containing `count` elements each, where
each new chunk starts `step` elements into the `enumerable`.

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

    iex> Enum.chunk_every([1, 2, 3, 4, 5, 6], 2)
    [[1, 2], [3, 4], [5, 6]]
    
    iex> Enum.chunk_every([1, 2, 3, 4, 5, 6], 3, 2, :discard)
    [[1, 2, 3], [3, 4, 5]]
    
    iex> Enum.chunk_every([1, 2, 3, 4, 5, 6], 3, 2, [7])
    [[1, 2, 3], [3, 4, 5], [5, 6, 7]]
    
    iex> Enum.chunk_every([1, 2, 3, 4], 3, 3, [])
    [[1, 2, 3], [4]]
    
    iex> Enum.chunk_every([1, 2, 3, 4], 10)
    [[1, 2, 3, 4]]
    
    iex> Enum.chunk_every([1, 2, 3, 4, 5], 2, 3, [])
    [[1, 2], [4, 5]]
    
    iex> Enum.chunk_every([1, 2, 3, 4], 3, 3, Stream.cycle([0]))
    [[1, 2, 3], [4, 0, 0]]

### chunk_while(enumerable, acc, chunk_fun, after_fun)
*(since 1.5.0)* 
```elixir
@spec chunk_while(
  t(),
  acc(),
  (element(), acc() -&gt; {:cont, chunk, acc()} | {:cont, acc()} | {:halt, acc()}),
  (acc() -&gt; {:cont, chunk, acc()} | {:cont, acc()})
) :: Enumerable.t()
when chunk: any()
```

Chunks the `enumerable` with fine grained control when every chunk is emitted.

`chunk_fun` receives the current element and the accumulator and must return:

- `{:cont, chunk, acc}` to emit a chunk and continue with the accumulator
- `{:cont, acc}` to not emit any chunk and continue with the accumulator
- `{:halt, acc}` to halt chunking over the `enumerable`.

`after_fun` is invoked with the final accumulator when iteration is
finished (or `halt`ed) to handle any trailing elements that were returned
as part of an accumulator, but were not emitted as a chunk by `chunk_fun`.
It must return:

- `{:cont, chunk, acc}` to emit a chunk. The chunk will be appended to the
  list of already emitted chunks.
- `{:cont, acc}` to not emit a chunk

The `acc` in `after_fun` is required in order to mirror the tuple format
from `chunk_fun` but it will be discarded since the traversal is complete.

Returns a list of emitted chunks.

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
    iex> Enum.chunk_while(1..10, [], chunk_fun, after_fun)
    [[1, 2], [3, 4], [5, 6], [7, 8], [9, 10]]
    iex> Enum.chunk_while([1, 2, 3, 5, 7], [], chunk_fun, after_fun)
    [[1, 2], [3, 5, 7]]

### concat(enumerables)

```elixir
@spec concat(t()) :: t()
```

Given an enumerable of enumerables, concatenates the `enumerables` into
a single list.

#### Examples

    iex> Enum.concat([1..3, 4..6, 7..9])
    [1, 2, 3, 4, 5, 6, 7, 8, 9]
    
    iex> Enum.concat([[1, [2], 3], [4], [5, 6]])
    [1, [2], 3, 4, 5, 6]

### concat(left, right)

```elixir
@spec concat(t(), t()) :: t()
```

Concatenates the enumerable on the `right` with the enumerable on the
`left`.

This function produces the same result as the `++/2` operator
for lists.

#### Examples

    iex> Enum.concat(1..3, 4..6)
    [1, 2, 3, 4, 5, 6]
    
    iex> Enum.concat([1, 2, 3], [4, 5, 6])
    [1, 2, 3, 4, 5, 6]

### count(enumerable)

```elixir
@spec count(t()) :: non_neg_integer()
```

Returns the size of the `enumerable`.

#### Examples

    iex> Enum.count([1, 2, 3])
    3

### count(enumerable, fun)

```elixir
@spec count(t(), (element() -&gt; as_boolean(term()))) :: non_neg_integer()
```

Returns the count of elements in the `enumerable` for which `fun` returns
a truthy value.

#### Examples

    iex> Enum.count([1, 2, 3, 4, 5], fn x -> rem(x, 2) == 0 end)
    2

### count_until(enumerable, limit)
*(since 1.12.0)* 
```elixir
@spec count_until(t(), pos_integer()) :: non_neg_integer()
```

Counts the enumerable stopping at `limit`.

This is useful for checking certain properties of the count of an enumerable
without having to actually count the entire enumerable. For example, if you
wanted to check that the count was exactly, at least, or more than a value.

If the enumerable implements `c:Enumerable.count/1`, the enumerable is
not traversed and we return the lower of the two numbers. To force
enumeration, use `count_until/3` with `fn _ -> true end` as the second
argument.

#### Examples

    iex> Enum.count_until(1..20, 5)
    5
    iex> Enum.count_until(1..20, 50)
    20
    iex> Enum.count_until(1..10, 10) == 10 # At least 10
    true
    iex> Enum.count_until(1..11, 10 + 1) > 10 # More than 10
    true
    iex> Enum.count_until(1..5, 10) < 10 # Less than 10
    true
    iex> Enum.count_until(1..10, 10 + 1) == 10 # Exactly ten
    true

### count_until(enumerable, fun, limit)
*(since 1.12.0)* 
```elixir
@spec count_until(t(), (element() -&gt; as_boolean(term())), pos_integer()) ::
  non_neg_integer()
```

Counts the elements in the enumerable for which `fun` returns a truthy value, stopping at `limit`.

See `count/2` and `count_until/2` for more information.

#### Examples

    iex> Enum.count_until(1..20, fn x -> rem(x, 2) == 0 end, 7)
    7
    iex> Enum.count_until(1..20, fn x -> rem(x, 2) == 0 end, 11)
    10

### dedup(enumerable)

```elixir
@spec dedup(t()) :: list()
```

Enumerates the `enumerable`, returning a list where all consecutive
duplicate elements are collapsed to a single element.

Elements are compared using `===/2`.

If you want to remove all duplicate elements, regardless of order,
see `uniq/1`.

#### Examples

    iex> Enum.dedup([1, 2, 3, 3, 2, 1])
    [1, 2, 3, 2, 1]
    
    iex> Enum.dedup([1, 1, 2, 2.0, :three, :three])
    [1, 2, 2.0, :three]

### dedup_by(enumerable, fun)

```elixir
@spec dedup_by(t(), (element() -&gt; term())) :: list()
```

Enumerates the `enumerable`, returning a list where all consecutive
duplicate elements are collapsed to a single element.

The function `fun` maps every element to a term which is used to
determine if two elements are duplicates.

#### Examples

    iex> Enum.dedup_by([{1, :a}, {2, :b}, {2, :c}, {1, :a}], fn {x, _} -> x end)
    [{1, :a}, {2, :b}, {1, :a}]
    
    iex> Enum.dedup_by([5, 1, 2, 3, 2, 1], fn x -> x > 2 end)
    [5, 1, 3, 2]

### drop(enumerable, amount)

```elixir
@spec drop(t(), integer()) :: list()
```

Drops the `amount` of elements from the `enumerable`.

If a negative `amount` is given, the `amount` of last values will be dropped.
The `enumerable` will be enumerated once to retrieve the proper index and
the remaining calculation is performed from the end.

#### Examples

    iex> Enum.drop([1, 2, 3], 2)
    [3]
    
    iex> Enum.drop([1, 2, 3], 10)
    []
    
    iex> Enum.drop([1, 2, 3], 0)
    [1, 2, 3]
    
    iex> Enum.drop([1, 2, 3], -1)
    [1, 2]

### drop_every(enumerable, nth)

```elixir
@spec drop_every(t(), non_neg_integer()) :: list()
```

Returns a list of every `nth` element in the `enumerable` dropped,
starting with the first element.

The first element is always dropped, unless `nth` is 0.

The second argument specifying every `nth` element must be a non-negative
integer.

#### Examples

    iex> Enum.drop_every(1..10, 2)
    [2, 4, 6, 8, 10]
    
    iex> Enum.drop_every(1..10, 0)
    [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    
    iex> Enum.drop_every([1, 2, 3], 1)
    []

### drop_while(enumerable, fun)

```elixir
@spec drop_while(t(), (element() -&gt; as_boolean(term()))) :: list()
```

Drops elements at the beginning of the `enumerable` while `fun` returns a
truthy value.

#### Examples

    iex> Enum.drop_while([1, 2, 3, 2, 1], fn x -> x < 3 end)
    [3, 2, 1]

### each(enumerable, fun)

```elixir
@spec each(t(), (element() -&gt; any())) :: :ok
```

Invokes the given `fun` for each element in the `enumerable`.

Returns `:ok`.

#### Examples

    Enum.each(["some", "example"], fn x -> IO.puts(x) end)
    "some"
    "example"
    #=> :ok

### empty?(enumerable)

```elixir
@spec empty?(t()) :: boolean()
```

Determines if the `enumerable` is empty.

Returns `true` if `enumerable` is empty, otherwise `false`.

#### Examples

    iex> Enum.empty?([])
    true
    
    iex> Enum.empty?([1, 2, 3])
    false

### fetch(enumerable, index)

```elixir
@spec fetch(t(), index()) :: {:ok, element()} | :error
```

Finds the element at the given `index` (zero-based).

Returns `{:ok, element}` if found, otherwise `:error`.

A negative `index` can be passed, which means the `enumerable` is
enumerated once and the `index` is counted from the end (for example,
`-1` fetches the last element).

#### Examples

    iex> Enum.fetch([2, 4, 6], 0)
    {:ok, 2}
    
    iex> Enum.fetch([2, 4, 6], -3)
    {:ok, 2}
    
    iex> Enum.fetch([2, 4, 6], 2)
    {:ok, 6}
    
    iex> Enum.fetch([2, 4, 6], 4)
    :error

### fetch!(enumerable, index)

```elixir
@spec fetch!(t(), index()) :: element()
```

Finds the element at the given `index` (zero-based).

Raises `OutOfBoundsError` if the given `index` is outside the range of
the `enumerable`.

#### Examples

    iex> Enum.fetch!([2, 4, 6], 0)
    2
    
    iex> Enum.fetch!([2, 4, 6], 2)
    6
    
    iex> Enum.fetch!([2, 4, 6], 4)
    ** (Enum.OutOfBoundsError) out of bounds error

### filter(enumerable, fun)

```elixir
@spec filter(t(), (element() -&gt; as_boolean(term()))) :: list()
```

Filters the `enumerable`, i.e. returns only those elements
for which `fun` returns a truthy value.

See also `reject/2` which discards all elements where the
function returns a truthy value.

#### Examples

    iex> Enum.filter([1, 2, 3], fn x -> rem(x, 2) == 0 end)
    [2]
    iex> Enum.filter(["apple", "pear", "banana"], fn fruit -> String.contains?(fruit, "a") end)
    ["apple", "pear", "banana"]
    iex> Enum.filter([4, 21, 24, 904], fn seconds -> seconds > 1000 end)
    []

Keep in mind that `filter` is not capable of filtering and
transforming an element at the same time. If you would like
to do so, consider using `flat_map/2`. For example, if you
want to convert all strings that represent an integer and
discard the invalid one in one pass:

    strings = ["1234", "abc", "12ab"]
    
    Enum.flat_map(strings, fn string ->
      case Integer.parse(string) do
        # transform to integer
        {int, _rest} -> [int]
        # skip the value
        :error -> []
      end
    end)

### find(enumerable, default \\ nil, fun)

```elixir
@spec find(t(), default(), (element() -&gt; any())) :: element() | default()
```

Returns the first element for which `fun` returns a truthy value.
If no such element is found, returns `default`.

#### Examples

    iex> Enum.find([2, 3, 4], fn x -> rem(x, 2) == 1 end)
    3
    
    iex> Enum.find([2, 4, 6], fn x -> rem(x, 2) == 1 end)
    nil
    iex> Enum.find([2, 4, 6], 0, fn x -> rem(x, 2) == 1 end)
    0

### find_index(enumerable, fun)

```elixir
@spec find_index(t(), (element() -&gt; any())) :: non_neg_integer() | nil
```

Similar to `find/3`, but returns the index (zero-based)
of the element instead of the element itself.

#### Examples

    iex> Enum.find_index([2, 4, 6], fn x -> rem(x, 2) == 1 end)
    nil
    
    iex> Enum.find_index([2, 3, 4], fn x -> rem(x, 2) == 1 end)
    1

### find_value(enumerable, default \\ nil, fun)

```elixir
@spec find_value(t(), default(), (element() -&gt; found_value)) ::
  found_value | default()
when found_value: term()
```

Similar to `find/3`, but returns the value of the function
invocation instead of the element itself.

The return value is considered to be found when the result is truthy
(neither `nil` nor `false`).

#### Examples

    iex> Enum.find_value([2, 3, 4], fn x ->
    ...>   if x > 2, do: x * x
    ...> end)
    9
    
    iex> Enum.find_value([2, 4, 6], fn x -> rem(x, 2) == 1 end)
    nil
    
    iex> Enum.find_value([2, 3, 4], fn x -> rem(x, 2) == 1 end)
    true
    
    iex> Enum.find_value([1, 2, 3], "no bools!", &is_boolean/1)
    "no bools!"

### flat_map(enumerable, fun)

```elixir
@spec flat_map(t(), (element() -&gt; t())) :: list()
```

Maps the given `fun` over `enumerable` and flattens the result.

This function returns a new enumerable built by appending the result of invoking `fun`
on each element of `enumerable` together; conceptually, this is similar to a
combination of `map/2` and `concat/1`.

#### Examples

    iex> Enum.flat_map([:a, :b, :c], fn x -> [x, x] end)
    [:a, :a, :b, :b, :c, :c]
    
    iex> Enum.flat_map([{1, 3}, {4, 6}], fn {x, y} -> x..y end)
    [1, 2, 3, 4, 5, 6]
    
    iex> Enum.flat_map([:a, :b, :c], fn x -> [[x]] end)
    [[:a], [:b], [:c]]

This is frequently used to to transform and filter in one pass, returning empty
lists to exclude results:

    iex> Enum.flat_map([4, 0, 2, 0], fn x ->
    ...>   if x != 0, do: [1 / x], else: []
    ...> end)
    [0.25, 0.5]

### flat_map_reduce(enumerable, acc, fun)

```elixir
@spec flat_map_reduce(t(), acc(), fun) :: {[any()], acc()}
when fun: (element(), acc() -&gt; {t(), acc()} | {:halt, acc()})
```

Maps and reduces an `enumerable`, flattening the given results (only one level deep).

It expects an accumulator and a function that receives each enumerable
element, and must return a tuple containing a new enumerable (often a list)
with the new accumulator or a tuple with `:halt` as first element and
the accumulator as second.

#### Examples

    iex> enumerable = 1..100
    iex> n = 3
    iex> Enum.flat_map_reduce(enumerable, 0, fn x, acc ->
    ...>   if acc < n, do: {[x], acc + 1}, else: {:halt, acc}
    ...> end)
    {[1, 2, 3], 3}
    
    iex> Enum.flat_map_reduce(1..5, 0, fn x, acc -> {[[x]], acc + x} end)
    {[[1], [2], [3], [4], [5]], 15}

### frequencies(enumerable)
*(since 1.10.0)* 
```elixir
@spec frequencies(t()) :: map()
```

Returns a map with keys as unique elements of `enumerable` and values
as the count of every element.

#### Examples

    iex> Enum.frequencies(~w{ant buffalo ant ant buffalo dingo})
    %{"ant" => 3, "buffalo" => 2, "dingo" => 1}

### frequencies_by(enumerable, key_fun)
*(since 1.10.0)* 
```elixir
@spec frequencies_by(t(), (element() -&gt; any())) :: map()
```

Returns a map with keys as unique elements given by `key_fun` and values
as the count of every element.

#### Examples

    iex> Enum.frequencies_by(~w{aa aA bb cc}, &String.downcase/1)
    %{"aa" => 2, "bb" => 1, "cc" => 1}
    
    iex> Enum.frequencies_by(~w{aaa aA bbb cc c}, &String.length/1)
    %{3 => 2, 2 => 2, 1 => 1}

### group_by(enumerable, key_fun, value_fun \\ fn x -&gt; x end)

```elixir
@spec group_by(t(), (element() -&gt; any()), (element() -&gt; any())) :: map()
```

Splits the `enumerable` into groups based on `key_fun`.

The result is a map where each key is given by `key_fun`
and each value is a list of elements given by `value_fun`.
The order of elements within each list is preserved from the `enumerable`.
However, like all maps, the resulting map is unordered.

#### Examples

    iex> Enum.group_by(~w{ant buffalo cat dingo}, &String.length/1)
    %{3 => ["ant", "cat"], 5 => ["dingo"], 7 => ["buffalo"]}
    
    iex> Enum.group_by(~w{ant buffalo cat dingo}, &String.length/1, &String.first/1)
    %{3 => ["a", "c"], 5 => ["d"], 7 => ["b"]}

The key can be any Elixir value. For example, you may use a tuple
to group by multiple keys:

    iex> collection = [
    ...>   %{id: 1, lang: "Elixir", seq: 1},
    ...>   %{id: 1, lang: "Java", seq: 1},
    ...>   %{id: 1, lang: "Ruby", seq: 2},
    ...>   %{id: 2, lang: "Python", seq: 1},
    ...>   %{id: 2, lang: "C#", seq: 2},
    ...>   %{id: 2, lang: "Haskell", seq: 2},
    ...> ]
    iex> Enum.group_by(collection, &{&1.id, &1.seq})
    %{
      {1, 1} => [%{id: 1, lang: "Elixir", seq: 1}, %{id: 1, lang: "Java", seq: 1}],
      {1, 2} => [%{id: 1, lang: "Ruby", seq: 2}],
      {2, 1} => [%{id: 2, lang: "Python", seq: 1}],
      {2, 2} => [%{id: 2, lang: "C#", seq: 2}, %{id: 2, lang: "Haskell", seq: 2}]
    }
    iex> Enum.group_by(collection, &{&1.id, &1.seq}, &{&1.id, &1.lang})
    %{
      {1, 1} => [{1, "Elixir"}, {1, "Java"}],
      {1, 2} => [{1, "Ruby"}],
      {2, 1} => [{2, "Python"}],
      {2, 2} => [{2, "C#"}, {2, "Haskell"}]
    }

### intersperse(enumerable, separator)

```elixir
@spec intersperse(t(), element()) :: list()
```

Intersperses `separator` between each element of the enumeration.

#### Examples

    iex> Enum.intersperse([1, 2, 3], 0)
    [1, 0, 2, 0, 3]
    
    iex> Enum.intersperse([1], 0)
    [1]
    
    iex> Enum.intersperse([], 0)
    []

### into(enumerable, collectable)

```elixir
@spec into(Enumerable.t(), Collectable.t()) :: Collectable.t()
```

Inserts the given `enumerable` into a `collectable`.

Note that passing a non-empty list as the `collectable` is deprecated.
If you're collecting into a non-empty keyword list, consider using
`Keyword.merge(collectable, Enum.to_list(enumerable))`. If you're collecting
into a non-empty list, consider something like `Enum.to_list(enumerable) ++ collectable`.

#### Examples

    iex> Enum.into([1, 2], [])
    [1, 2]
    
    iex> Enum.into([a: 1, b: 2], %{})
    %{a: 1, b: 2}
    
    iex> Enum.into(%{a: 1}, %{b: 2})
    %{a: 1, b: 2}
    
    iex> Enum.into([a: 1, a: 2], %{})
    %{a: 2}
    
    iex> Enum.into([a: 2], %{a: 1, b: 3})
    %{a: 2, b: 3}

### into(enumerable, collectable, transform)

```elixir
@spec into(Enumerable.t(), Collectable.t(), (term() -&gt; term())) :: Collectable.t()
```

Inserts the given `enumerable` into a `collectable` according to the
transformation function.

#### Examples

    iex> Enum.into([1, 2, 3], [], fn x -> x * 3 end)
    [3, 6, 9]
    
    iex> Enum.into(%{a: 1, b: 2}, %{c: 3}, fn {k, v} -> {k, v * 2} end)
    %{a: 2, b: 4, c: 3}

### join(enumerable, joiner \\ &quot;&quot;)

```elixir
@spec join(t(), binary()) :: binary()
```

Joins the given `enumerable` into a string using `joiner` as a
separator.

If `joiner` is not passed at all, it defaults to an empty string.

All elements in the `enumerable` must be convertible to a string
or be a binary, otherwise an error is raised.

#### Examples

    iex> Enum.join([1, 2, 3])
    "123"
    
    iex> Enum.join([1, 2, 3], " = ")
    "1 = 2 = 3"
    
    iex> Enum.join([["a", "b"], ["c", "d", "e", ["f", "g"]], "h", "i"], " ")
    "ab cdefg h i"

### map(enumerable, fun)

```elixir
@spec map(t(), (element() -&gt; any())) :: list()
```

Returns a list where each element is the result of invoking
`fun` on each corresponding element of `enumerable`.

For maps, the function expects a key-value tuple.

#### Examples

    iex> Enum.map([1, 2, 3], fn x -> x * 2 end)
    [2, 4, 6]
    
    iex> Enum.map([a: 1, b: 2], fn {k, v} -> {k, -v} end)
    [a: -1, b: -2]

### map_every(enumerable, nth, fun)
*(since 1.4.0)* 
```elixir
@spec map_every(t(), non_neg_integer(), (element() -&gt; any())) :: list()
```

Returns a list of results of invoking `fun` on every `nth`
element of `enumerable`, starting with the first element.

The first element is always passed to the given function, unless `nth` is `0`.

The second argument specifying every `nth` element must be a non-negative
integer.

If `nth` is `0`, then `enumerable` is directly converted to a list,
without `fun` being ever applied.

#### Examples

    iex> Enum.map_every(1..10, 2, fn x -> x + 1000 end)
    [1001, 2, 1003, 4, 1005, 6, 1007, 8, 1009, 10]
    
    iex> Enum.map_every(1..10, 3, fn x -> x + 1000 end)
    [1001, 2, 3, 1004, 5, 6, 1007, 8, 9, 1010]
    
    iex> Enum.map_every(1..5, 0, fn x -> x + 1000 end)
    [1, 2, 3, 4, 5]
    
    iex> Enum.map_every([1, 2, 3], 1, fn x -> x + 1000 end)
    [1001, 1002, 1003]

### map_intersperse(enumerable, separator, mapper)
*(since 1.10.0)* 
```elixir
@spec map_intersperse(t(), element(), (element() -&gt; any())) :: list()
```

Maps and intersperses the given enumerable in one pass.

#### Examples

    iex> Enum.map_intersperse([1, 2, 3], :a, &(&1 * 2))
    [2, :a, 4, :a, 6]

### map_join(enumerable, joiner \\ &quot;&quot;, mapper)

```elixir
@spec map_join(t(), String.t(), (element() -&gt; String.Chars.t())) :: String.t()
```

Maps and joins the given `enumerable` in one pass.

If `joiner` is not passed at all, it defaults to an empty string.

All elements returned from invoking the `mapper` must be convertible to
a string, otherwise an error is raised.

#### Examples

    iex> Enum.map_join([1, 2, 3], &(&1 * 2))
    "246"
    
    iex> Enum.map_join([1, 2, 3], " = ", &(&1 * 2))
    "2 = 4 = 6"

### map_reduce(enumerable, acc, fun)

```elixir
@spec map_reduce(t(), acc(), (element(), acc() -&gt; {element(), acc()})) ::
  {list(), acc()}
```

Invokes the given function to each element in the `enumerable` to reduce
it to a single element, while keeping an accumulator.

Returns a tuple where the first element is the mapped enumerable and
the second one is the final accumulator.

The function, `fun`, receives two arguments: the first one is the
element, and the second one is the accumulator. `fun` must return
a tuple with two elements in the form of `{result, accumulator}`.

For maps, the first tuple element must be a `{key, value}` tuple.

#### Examples

    iex> Enum.map_reduce([1, 2, 3], 0, fn x, acc -> {x * 2, x + acc} end)
    {[2, 4, 6], 6}

### max(enumerable, sorter \\ &amp;&gt;=/2, empty_fallback \\ fn -&gt; raise Enum.EmptyError end)

```elixir
@spec max(t(), (element(), element() -&gt; boolean()) | module(), (-&gt; empty_result)) ::
  element() | empty_result
when empty_result: any()
```

Returns the maximal element in the `enumerable` according
to Erlang's term ordering.

By default, the comparison is done with the `>=` sorter function.
If multiple elements are considered maximal, the first one that
was found is returned. If you want the last element considered
maximal to be returned, the sorter function should not return true
for equal elements.

If the enumerable is empty, the provided `empty_fallback` is called.
The default `empty_fallback` raises `Enum.EmptyError`.

#### Examples

    iex> Enum.max([1, 2, 3])
    3

The fact this function uses Erlang's term ordering means that the comparison
is structural and not semantic. For example:

    iex> Enum.max([~D[2017-03-31], ~D[2017-04-01]])
    ~D[2017-03-31]

In the example above, `max/2` returned March 31st instead of April 1st
because the structural comparison compares the day before the year.
For this reason, most structs provide a "compare" function, such as
`Date.compare/2`, which receives two structs and returns `:lt` (less-than),
`:eq` (equal to), and `:gt` (greater-than). If you pass a module as the
sorting function, Elixir will automatically use the `compare/2` function
of said module:

    iex> Enum.max([~D[2017-03-31], ~D[2017-04-01]], Date)
    ~D[2017-04-01]

Finally, if you don't want to raise on empty enumerables, you can pass
the empty fallback:

    iex> Enum.max([], &>=/2, fn -> 0 end)
    0

### max_by(enumerable, fun, sorter \\ &amp;&gt;=/2, empty_fallback \\ fn -&gt; raise Enum.EmptyError end)

```elixir
@spec max_by(
  t(),
  (element() -&gt; any()),
  (element(), element() -&gt; boolean()) | module(),
  (-&gt; empty_result)
) :: element() | empty_result
when empty_result: any()
```

Returns the maximal element in the `enumerable` as calculated
by the given `fun`.

By default, the comparison is done with the `>=` sorter function.
If multiple elements are considered maximal, the first one that
was found is returned. If you want the last element considered
maximal to be returned, the sorter function should not return true
for equal elements.

Calls the provided `empty_fallback` function and returns its value if
`enumerable` is empty. The default `empty_fallback` raises `Enum.EmptyError`.

#### Examples

    iex> Enum.max_by(["a", "aa", "aaa"], fn x -> String.length(x) end)
    "aaa"
    
    iex> Enum.max_by(["a", "aa", "aaa", "b", "bbb"], &String.length/1)
    "aaa"

The fact this function uses Erlang's term ordering means that the
comparison is structural and not semantic. Therefore, if you want
to compare structs, most structs provide a "compare" function, such as
`Date.compare/2`, which receives two structs and returns `:lt` (less-than),
`:eq` (equal to), and `:gt` (greater-than). If you pass a module as the
sorting function, Elixir will automatically use the `compare/2` function
of said module:

    iex> users = [
    ...>   %{name: "Ellis", birthday: ~D[1943-05-11]},
    ...>   %{name: "Lovelace", birthday: ~D[1815-12-10]},
    ...>   %{name: "Turing", birthday: ~D[1912-06-23]}
    ...> ]
    iex> Enum.max_by(users, &(&1.birthday), Date)
    %{name: "Ellis", birthday: ~D[1943-05-11]}

Finally, if you don't want to raise on empty enumerables, you can pass
the empty fallback:

    iex> Enum.max_by([], &String.length/1, fn -> nil end)
    nil

### member?(enumerable, element)

```elixir
@spec member?(t(), element()) :: boolean()
```

Checks if `element` exists within the `enumerable`.

Membership is tested with the match (`===/2`) operator.

#### Examples

    iex> Enum.member?(1..10, 5)
    true
    iex> Enum.member?(1..10, 5.0)
    false
    
    iex> Enum.member?([1.0, 2.0, 3.0], 2)
    false
    iex> Enum.member?([1.0, 2.0, 3.0], 2.000)
    true
    
    iex> Enum.member?([:a, :b, :c], :d)
    false

When called outside guards, the [`in`](\`in/2\`) and [`not in`](\`in/2\`)
operators work by using this function.

### min(enumerable, sorter \\ &amp;&lt;=/2, empty_fallback \\ fn -&gt; raise Enum.EmptyError end)

```elixir
@spec min(t(), (element(), element() -&gt; boolean()) | module(), (-&gt; empty_result)) ::
  element() | empty_result
when empty_result: any()
```

Returns the minimal element in the `enumerable` according
to Erlang's term ordering.

By default, the comparison is done with the `<=` sorter function.
If multiple elements are considered minimal, the first one that
was found is returned. If you want the last element considered
minimal to be returned, the sorter function should not return true
for equal elements.

If the enumerable is empty, the provided `empty_fallback` is called.
The default `empty_fallback` raises `Enum.EmptyError`.

#### Examples

    iex> Enum.min([1, 2, 3])
    1

The fact this function uses Erlang's term ordering means that the comparison
is structural and not semantic. For example:

    iex> Enum.min([~D[2017-03-31], ~D[2017-04-01]])
    ~D[2017-04-01]

In the example above, `min/2` returned April 1st instead of March 31st
because the structural comparison compares the day before the year.
For this reason, most structs provide a "compare" function, such as
`Date.compare/2`, which receives two structs and returns `:lt` (less-than),
`:eq` (equal to), and `:gt` (greater-than). If you pass a module as the
sorting function, Elixir will automatically use the `compare/2` function
of said module:

    iex> Enum.min([~D[2017-03-31], ~D[2017-04-01]], Date)
    ~D[2017-03-31]

Finally, if you don't want to raise on empty enumerables, you can pass
the empty fallback:

    iex> Enum.min([], fn -> 0 end)
    0

### min_by(enumerable, fun, sorter \\ &amp;&lt;=/2, empty_fallback \\ fn -&gt; raise Enum.EmptyError end)

```elixir
@spec min_by(
  t(),
  (element() -&gt; any()),
  (element(), element() -&gt; boolean()) | module(),
  (-&gt; empty_result)
) :: element() | empty_result
when empty_result: any()
```

Returns the minimal element in the `enumerable` as calculated
by the given `fun`.

By default, the comparison is done with the `<=` sorter function.
If multiple elements are considered minimal, the first one that
was found is returned. If you want the last element considered
minimal to be returned, the sorter function should not return true
for equal elements.

Calls the provided `empty_fallback` function and returns its value if
`enumerable` is empty. The default `empty_fallback` raises `Enum.EmptyError`.

#### Examples

    iex> Enum.min_by(["a", "aa", "aaa"], fn x -> String.length(x) end)
    "a"
    
    iex> Enum.min_by(["a", "aa", "aaa", "b", "bbb"], &String.length/1)
    "a"

The fact this function uses Erlang's term ordering means that the
comparison is structural and not semantic. Therefore, if you want
to compare structs, most structs provide a "compare" function, such as
`Date.compare/2`, which receives two structs and returns `:lt` (less-than),
`:eq` (equal to), and `:gt` (greater-than). If you pass a module as the
sorting function, Elixir will automatically use the `compare/2` function
of said module:

    iex> users = [
    ...>   %{name: "Ellis", birthday: ~D[1943-05-11]},
    ...>   %{name: "Lovelace", birthday: ~D[1815-12-10]},
    ...>   %{name: "Turing", birthday: ~D[1912-06-23]}
    ...> ]
    iex> Enum.min_by(users, &(&1.birthday), Date)
    %{name: "Lovelace", birthday: ~D[1815-12-10]}

Finally, if you don't want to raise on empty enumerables, you can pass
the empty fallback:

    iex> Enum.min_by([], &String.length/1, fn -> nil end)
    nil

### min_max(enumerable, empty_fallback \\ fn -&gt; raise Enum.EmptyError end)

```elixir
@spec min_max(t(), (-&gt; empty_result)) :: {element(), element()} | empty_result
when empty_result: any()
```

Returns a tuple with the minimal and the maximal elements in the
enumerable according to Erlang's term ordering.

If multiple elements are considered maximal or minimal, the first one
that was found is returned.

Calls the provided `empty_fallback` function and returns its value if
`enumerable` is empty. The default `empty_fallback` raises `Enum.EmptyError`.

#### Examples

    iex> Enum.min_max([2, 3, 1])
    {1, 3}
    
    iex> Enum.min_max([], fn -> {nil, nil} end)
    {nil, nil}

### min_max_by(enumerable, fun, sorter_or_empty_fallback \\ &amp;&lt;/2, empty_fallback \\ fn -&gt; raise Enum.EmptyError end)

```elixir
@spec min_max_by(
  t(),
  (element() -&gt; any()),
  (element(), element() -&gt; boolean()) | module(),
  (-&gt; empty_result)
) :: {element(), element()} | empty_result
when empty_result: any()
```

Returns a tuple with the minimal and the maximal elements in the
enumerable as calculated by the given function.

If multiple elements are considered maximal or minimal, the first one
that was found is returned.

#### Examples

    iex> Enum.min_max_by(["aaa", "bb", "c"], fn x -> String.length(x) end)
    {"c", "aaa"}
    
    iex> Enum.min_max_by(["aaa", "a", "bb", "c", "ccc"], &String.length/1)
    {"a", "aaa"}
    
    iex> Enum.min_max_by([], &String.length/1, fn -> {nil, nil} end)
    {nil, nil}

The fact this function uses Erlang's term ordering means that the
comparison is structural and not semantic. Therefore, if you want
to compare structs, most structs provide a "compare" function, such as
`Date.compare/2`, which receives two structs and returns `:lt` (less-than),
`:eq` (equal to), and `:gt` (greater-than). If you pass a module as the
sorting function, Elixir will automatically use the `compare/2` function
of said module:

    iex> users = [
    ...>   %{name: "Ellis", birthday: ~D[1943-05-11]},
    ...>   %{name: "Lovelace", birthday: ~D[1815-12-10]},
    ...>   %{name: "Turing", birthday: ~D[1912-06-23]}
    ...> ]
    iex> Enum.min_max_by(users, &(&1.birthday), Date)
    {
      %{name: "Lovelace", birthday: ~D[1815-12-10]},
      %{name: "Ellis", birthday: ~D[1943-05-11]}
    }

Finally, if you don't want to raise on empty enumerables, you can pass
the empty fallback:

    iex> Enum.min_max_by([], &String.length/1, fn -> nil end)
    nil

### product(enumerable)
*(since 1.12.0)* 
```elixir
@spec product(t()) :: number()
```

Returns the product of all elements.

Raises `ArithmeticError` if `enumerable` contains a non-numeric value.

If you need to apply a transformation first, consider using `Enum.product_by/2` instead.

#### Examples

    iex> Enum.product([])
    1
    iex> Enum.product([2, 3, 4])
    24
    iex> Enum.product([2.0, 3.0, 4.0])
    24.0

### product_by(enumerable, mapper)
*(since 1.18.0)* 
```elixir
@spec product_by(t(), (element() -&gt; number())) :: number()
```

Maps and computes the product of the given `enumerable` in one pass.

Raises `ArithmeticError` if `mapper` returns a non-numeric value.

#### Examples

    iex> Enum.product_by([%{count: 2}, %{count: 4}, %{count: 3}], fn x -> x.count end)
    24
    
    iex> Enum.product_by(1..3, fn x -> x ** 2 end)
    36
    
    iex> Enum.product_by([], fn x -> x.count end)
    1

Filtering can be achieved by returning `1` to ignore elements:

    iex> Enum.product_by([2, -1, 3], fn x -> if x > 0, do: x, else: 1 end)
    6

### random(enumerable)

```elixir
@spec random(t()) :: element()
```

Returns a random element of an `enumerable`.

Raises `Enum.EmptyError` if `enumerable` is empty.

This function uses Erlang's [`:rand` module](\`:rand\`) to calculate
the random value. Check its documentation for setting a
different random algorithm or a different seed.

If a range is passed into the function, this function will pick a
random value between the range limits, without traversing the whole
range (thus executing in constant time and constant memory).

#### Examples

The examples below use the `:exsss` pseudorandom algorithm since it's
the default from Erlang/OTP 22:

    # Although not necessary, let's seed the random algorithm
    iex> :rand.seed(:exsss, {100, 101, 102})
    iex> Enum.random([1, 2, 3])
    2
    iex> Enum.random([1, 2, 3])
    1
    iex> Enum.random(1..1_000)
    309

#### Implementation

The random functions in this module implement reservoir sampling,
which allows them to sample infinite collections. In particular,
we implement Algorithm L, as described in by Kim-Hung Li in
"Reservoir-Sampling Algorithms of Time Complexity O(n(1+log(N/n)))".

### reduce(enumerable, fun)

```elixir
@spec reduce(t(), (element(), acc() -&gt; acc())) :: acc()
```

Invokes `fun` for each element in the `enumerable` with the
accumulator.

Raises `Enum.EmptyError` if `enumerable` is empty.

The first element of the `enumerable` is used as the initial value
of the accumulator. Then, the function is invoked with the next
element and the accumulator. The result returned by the function
is used as the accumulator for the next iteration, recursively.
When the `enumerable` is done, the last accumulator is returned.

Since the first element of the enumerable is used as the initial
value of the accumulator, `fun` will only be executed `n - 1` times
where `n` is the length of the enumerable. This function won't call
the specified function for enumerables that are one-element long.

If you wish to use another value for the accumulator, use
`Enum.reduce/3`.

#### Examples

    iex> Enum.reduce([1, 2, 3, 4], fn x, acc -> x * acc end)
    24

### reduce(enumerable, acc, fun)

```elixir
@spec reduce(t(), acc(), (element(), acc() -&gt; acc())) :: acc()
```

Invokes `fun` for each element in the `enumerable` with the accumulator.

The initial value of the accumulator is `acc`. The function is invoked for
each element in the enumerable with the accumulator. The result returned
by the function is used as the accumulator for the next iteration.
The function returns the last accumulator.

#### Examples

    iex> Enum.reduce([1, 2, 3], 0, fn x, acc -> x + acc end)
    6
    
    iex> Enum.reduce(%{a: 2, b: 3, c: 4}, 0, fn {_key, val}, acc -> acc + val end)
    9

#### Reduce as a building block

Reduce (sometimes called `fold`) is a basic building block in functional
programming. Almost all of the functions in the `Enum` module can be
implemented on top of reduce. Those functions often rely on other operations,
such as `Enum.reverse/1`, which are optimized by the runtime.

For example, we could implement `map/2` in terms of `reduce/3` as follows:

    def my_map(enumerable, fun) do
      enumerable
      |> Enum.reduce([], fn x, acc -> [fun.(x) | acc] end)
      |> Enum.reverse()
    end

In the example above, `Enum.reduce/3` accumulates the result of each call
to `fun` into a list in reverse order, which is correctly ordered at the
end by calling `Enum.reverse/1`.

Implementing functions like `map/2`, `filter/2` and others are a good
exercise for understanding the power behind `Enum.reduce/3`. When an
operation cannot be expressed by any of the functions in the `Enum`
module, developers will most likely resort to `reduce/3`.

### reduce_while(enumerable, acc, fun)

```elixir
@spec reduce_while(t(), any(), (element(), any() -&gt; {:cont, any()} | {:halt, any()})) ::
  any()
```

Reduces `enumerable` until `fun` returns `{:halt, term}`.

The return value for `fun` is expected to be

- `{:cont, acc}` to continue the reduction with `acc` as the new
  accumulator or
- `{:halt, acc}` to halt the reduction

If `fun` returns `{:halt, acc}` the reduction is halted and the function
returns `acc`. Otherwise, if the enumerable is exhausted, the function returns
the accumulator of the last `{:cont, acc}`.

#### Examples

    iex> Enum.reduce_while(1..100, 0, fn x, acc ->
    ...>   if x < 5 do
    ...>     {:cont, acc + x}
    ...>   else
    ...>     {:halt, acc}
    ...>   end
    ...> end)
    10
    iex> Enum.reduce_while(1..100, 0, fn x, acc ->
    ...>   if x > 0 do
    ...>     {:cont, acc + x}
    ...>   else
    ...>     {:halt, acc}
    ...>   end
    ...> end)
    5050

### reject(enumerable, fun)

```elixir
@spec reject(t(), (element() -&gt; as_boolean(term()))) :: list()
```

Returns a list of elements in `enumerable` excluding those for which the function `fun` returns
a truthy value.

See also `filter/2`.

#### Examples

    iex> Enum.reject([1, 2, 3], fn x -> rem(x, 2) == 0 end)
    [1, 3]

### reverse(enumerable)

```elixir
@spec reverse(t()) :: list()
```

Returns a list of elements in `enumerable` in reverse order.

#### Examples

    iex> Enum.reverse([1, 2, 3])
    [3, 2, 1]

### reverse(enumerable, tail)

```elixir
@spec reverse(t(), t()) :: list()
```

Reverses the elements in `enumerable`, appends the `tail`, and returns
it as a list.

This is an optimization for
`enumerable |> Enum.reverse() |> Enum.concat(tail)`.

#### Examples

    iex> Enum.reverse([1, 2, 3], [4, 5, 6])
    [3, 2, 1, 4, 5, 6]

### reverse_slice(enumerable, start_index, count)

```elixir
@spec reverse_slice(t(), non_neg_integer(), non_neg_integer()) :: list()
```

Reverses the `enumerable` in the range from initial `start_index`
through `count` elements.

If `count` is greater than the size of the rest of the `enumerable`,
then this function will reverse the rest of the enumerable.

#### Examples

    iex> Enum.reverse_slice([1, 2, 3, 4, 5, 6], 2, 4)
    [1, 2, 6, 5, 4, 3]

### scan(enumerable, fun)

```elixir
@spec scan(t(), (element(), any() -&gt; any())) :: list()
```

Applies the given function to each element in the `enumerable`,
storing the result in a list and passing it as the accumulator
for the next computation. Uses the first element in the `enumerable`
as the starting value.

#### Examples

    iex> Enum.scan(1..5, &(&1 + &2))
    [1, 3, 6, 10, 15]

### scan(enumerable, acc, fun)

```elixir
@spec scan(t(), any(), (element(), any() -&gt; any())) :: list()
```

Applies the given function to each element in the `enumerable`,
storing the result in a list and passing it as the accumulator
for the next computation. Uses the given `acc` as the starting value.

#### Examples

    iex> Enum.scan(1..5, 0, &(&1 + &2))
    [1, 3, 6, 10, 15]

### shuffle(enumerable)

```elixir
@spec shuffle(t()) :: list()
```

Returns a list with the elements of `enumerable` shuffled.

This function uses Erlang's [`:rand` module](\`:rand\`) to calculate
the random value. Check its documentation for setting a
different random algorithm or a different seed.

#### Examples

The examples below use the `:exsss` pseudorandom algorithm since it's
the default from Erlang/OTP 22:

    # Although not necessary, let's seed the random algorithm
    iex> :rand.seed(:exsss, {11, 22, 33})
    iex> Enum.shuffle([1, 2, 3])
    [2, 1, 3]
    iex> Enum.shuffle([1, 2, 3])
    [2, 3, 1]

### slice(enumerable, index_range)
*(since 1.6.0)* 
```elixir
@spec slice(t(), Range.t()) :: list()
```

Returns a subset list of the given `enumerable` by `index_range`.

`index_range` must be a `Range`. Given an `enumerable`, it drops
elements before `index_range.first` (zero-base), then it takes elements
until element `index_range.last` (inclusively).

Indexes are normalized, meaning that negative indexes will be counted
from the end (for example, `-1` means the last element of the `enumerable`).

If `index_range.last` is out of bounds, then it is assigned as the index
of the last element.

If the normalized `index_range.first` is out of bounds of the given
`enumerable`, or this one is greater than the normalized `index_range.last`,
then `[]` is returned.

If a step `n` (other than `1`) is used in `index_range`, then it takes
every `n`th element from `index_range.first` to `index_range.last`
(according to the same rules described above).

#### Examples

    iex> Enum.slice([1, 2, 3, 4, 5], 1..3)
    [2, 3, 4]
    
    iex> Enum.slice([1, 2, 3, 4, 5], 3..10)
    [4, 5]
    
    # Last three elements (negative indexes)
    iex> Enum.slice([1, 2, 3, 4, 5], -3..-1)
    [3, 4, 5]

For ranges where `start > stop`, you need to explicit
mark them as increasing:

    iex> Enum.slice([1, 2, 3, 4, 5], 1..-2//1)
    [2, 3, 4]

The step can be any positive number. For example, to
get every 2 elements of the collection:

    iex> Enum.slice([1, 2, 3, 4, 5], 0..-1//2)
    [1, 3, 5]

To get every third element of the first ten elements:

    iex> integers = Enum.to_list(1..20)
    iex> Enum.slice(integers, 0..9//3)
    [1, 4, 7, 10]

If the first position is after the end of the enumerable
or after the last position of the range, it returns an
empty list:

    iex> Enum.slice([1, 2, 3, 4, 5], 6..10)
    []
    
    # first is greater than last
    iex> Enum.slice([1, 2, 3, 4, 5], 6..5//1)
    []

### slice(enumerable, start_index, amount)

```elixir
@spec slice(t(), index(), non_neg_integer()) :: list()
```

Returns a subset list of the given `enumerable`, from `start_index` (zero-based)
with `amount` number of elements if available.

Given an `enumerable`, it drops elements right before element `start_index`;
then, it takes `amount` of elements, returning as many elements as possible if
there are not enough elements.

A negative `start_index` can be passed, which means the `enumerable` is
enumerated once and the index is counted from the end (for example,
`-1` starts slicing from the last element).

It returns `[]` if `amount` is `0` or if `start_index` is out of bounds.

#### Examples

    iex> Enum.slice(1..100, 5, 10)
    [6, 7, 8, 9, 10, 11, 12, 13, 14, 15]
    
    # amount to take is greater than the number of elements
    iex> Enum.slice(1..10, 5, 100)
    [6, 7, 8, 9, 10]
    
    iex> Enum.slice(1..10, 5, 0)
    []
    
    # using a negative start index
    iex> Enum.slice(1..10, -6, 3)
    [5, 6, 7]
    iex> Enum.slice(1..10, -11, 5)
    [1, 2, 3, 4, 5]
    
    # out of bound start index
    iex> Enum.slice(1..10, 10, 5)
    []

### slide(enumerable, range_or_single_index, insertion_index)
*(since 1.13.0)* 
```elixir
@spec slide(t(), Range.t() | index(), index()) :: list()
```

Slides a single or multiple elements given by `range_or_single_index` from `enumerable`
to `insertion_index`.

The semantics of the range to be moved match the semantics of `Enum.slice/2`.
Specifically, that means:

- Indices are normalized, meaning that negative indexes will be counted from the end
  (for example, -1 means the last element of the enumerable). This will result in *two*
  traversals of your enumerable on types like lists that don't provide a constant-time count.

- If the normalized index range's `last` is out of bounds, the range is truncated to the last element.

- If the normalized index range's `first` is out of bounds, the selected range for sliding
  will be empty, so you'll get back your input list.

- Decreasing ranges (such as `5..0//1`) also select an empty range to be moved,
  so you'll get back your input list.

- Ranges with any step but 1 will raise an error.

#### Examples

    # Slide a single element
    iex> Enum.slide([:a, :b, :c, :d, :e, :f, :g], 5, 1)
    [:a, :f, :b, :c, :d, :e, :g]
    
    # Slide a range of elements towards the head of the list
    iex> Enum.slide([:a, :b, :c, :d, :e, :f, :g], 3..5, 1)
    [:a, :d, :e, :f, :b, :c, :g]
    
    # Slide a range of elements towards the tail of the list
    iex> Enum.slide([:a, :b, :c, :d, :e, :f, :g], 1..3, 5)
    [:a, :e, :f, :b, :c, :d, :g]
    
    # Slide with negative indices (counting from the end)
    iex> Enum.slide([:a, :b, :c, :d, :e, :f, :g], 3..-1//1, 2)
    [:a, :b, :d, :e, :f, :g, :c]
    iex> Enum.slide([:a, :b, :c, :d, :e, :f, :g], -4..-2, 1)
    [:a, :d, :e, :f, :b, :c, :g]
    
    # Insert at negative indices (counting from the end)
    iex> Enum.slide([:a, :b, :c, :d, :e, :f, :g], 3, -1)
    [:a, :b, :c, :e, :f, :g, :d]

### sort(enumerable)

```elixir
@spec sort(t()) :: list()
```

Sorts the `enumerable` according to Erlang's term ordering.

This function uses the merge sort algorithm. Do not use this
function to sort structs, see `sort/2` for more information.

#### Examples

    iex> Enum.sort([3, 2, 1])
    [1, 2, 3]

### sort(enumerable, sorter)

```elixir
@spec sort(
  t(),
  (element(), element() -&gt; boolean())
  | :asc
  | :desc
  | module()
  | {:asc | :desc, module()}
) :: list()
```

Sorts the `enumerable` by the given function.

This function uses the merge sort algorithm. The given function should compare
two arguments, and return `true` if the first argument precedes or is in the
same place as the second one.

#### Examples

    iex> Enum.sort([1, 2, 3], &(&1 >= &2))
    [3, 2, 1]

The sorting algorithm will be stable as long as the given function
returns `true` for values considered equal:

    iex> Enum.sort(["some", "kind", "of", "monster"], &(byte_size(&1) <= byte_size(&2)))
    ["of", "some", "kind", "monster"]

If the function does not return `true` for equal values, the sorting
is not stable and the order of equal terms may be shuffled.
For example:

    iex> Enum.sort(["some", "kind", "of", "monster"], &(byte_size(&1) < byte_size(&2)))
    ["of", "kind", "some", "monster"]

#### Ascending and descending (since v1.10.0)

`sort/2` allows a developer to pass `:asc` or `:desc` as the sorter, which is a convenience for
[`&<=/2`](\`\<=/2\`) and [`&>=/2`](\`\>=/2\`) respectively.

    iex> Enum.sort([2, 3, 1], :asc)
    [1, 2, 3]
    iex> Enum.sort([2, 3, 1], :desc)
    [3, 2, 1]

#### Sorting structs

Do not use `</2`, `<=/2`, `>/2`, `>=/2` and friends when sorting structs.
That's because the built-in operators above perform structural comparison
and not a semantic one. Imagine we sort the following list of dates:

    iex> dates = [~D[2019-01-01], ~D[2020-03-02], ~D[2019-06-06]]
    iex> Enum.sort(dates)
    [~D[2019-01-01], ~D[2020-03-02], ~D[2019-06-06]]

Note that the returned result is incorrect, because `sort/1` by default uses
`<=/2`, which will compare their structure. When comparing structures, the
fields are compared in alphabetical order, which means the dates above will
be compared by `day`, `month` and then `year`, which is the opposite of what
we want.

For this reason, most structs provide a "compare" function, such as
`Date.compare/2`, which receives two structs and returns `:lt` (less-than),
`:eq` (equal to), and `:gt` (greater-than). If you pass a module as the
sorting function, Elixir will automatically use the `compare/2` function
of said module:

    iex> dates = [~D[2019-01-01], ~D[2020-03-02], ~D[2019-06-06]]
    iex> Enum.sort(dates, Date)
    [~D[2019-01-01], ~D[2019-06-06], ~D[2020-03-02]]

To retrieve all dates in descending order, you can wrap the module in
a tuple with `:asc` or `:desc` as first element:

    iex> dates = [~D[2019-01-01], ~D[2020-03-02], ~D[2019-06-06]]
    iex> Enum.sort(dates, {:asc, Date})
    [~D[2019-01-01], ~D[2019-06-06], ~D[2020-03-02]]
    iex> dates = [~D[2019-01-01], ~D[2020-03-02], ~D[2019-06-06]]
    iex> Enum.sort(dates, {:desc, Date})
    [~D[2020-03-02], ~D[2019-06-06], ~D[2019-01-01]]

### sort_by(enumerable, mapper, sorter \\ :asc)

```elixir
@spec sort_by(
  t(),
  (element() -&gt; mapped_element),
  (element(), element() -&gt; boolean())
  | :asc
  | :desc
  | module()
  | {:asc | :desc, module()}
) :: list()
when mapped_element: element()
```

Sorts the mapped results of the `enumerable` according to the provided `sorter`
function.

This function maps each element of the `enumerable` using the
provided `mapper` function. The enumerable is then sorted by
the mapped elements using the `sorter`, which defaults to `:asc`
and sorts the elements ascendingly.

`sort_by/3` differs from `sort/2` in that it only calculates the
comparison value for each element in the enumerable once instead of
once for each element in each comparison. If the same function is
being called on both elements, it's more efficient to use `sort_by/3`.

#### Ascending and descending (since v1.10.0)

`sort_by/3` allows a developer to pass `:asc` or `:desc` as the sorter,
which is a convenience for [`&<=/2`](\`\<=/2\`) and [`&>=/2`](\`\>=/2\`) respectively:
iex\> Enum.sort\_by(\[2, 3, 1\], &(&1), :asc)
\[1, 2, 3\]

    iex> Enum.sort_by([2, 3, 1], &(&1), :desc)
    [3, 2, 1]

#### Examples

Using the default `sorter` of `:asc` :

    iex> Enum.sort_by(["some", "kind", "of", "monster"], &byte_size/1)
    ["of", "some", "kind", "monster"]

Sorting by multiple properties - first by size, then by first letter
(this takes advantage of the fact that tuples are compared element-by-element):

    iex> Enum.sort_by(["some", "kind", "of", "monster"], &{byte_size(&1), String.first(&1)})
    ["of", "kind", "some", "monster"]

Similar to `sort/2`, you can pass a custom sorter:

    iex> Enum.sort_by(["some", "kind", "of", "monster"], &byte_size/1, :desc)
    ["monster", "some", "kind", "of"]

As in `sort/2`, avoid using the default sorting function to sort
structs, as by default it performs structural comparison instead of
a semantic one. In such cases, you shall pass a sorting function as
third element or any module that implements a `compare/2` function.
For example, to sort users by their birthday in both ascending and
descending order respectively:

    iex> users = [
    ...>   %{name: "Ellis", birthday: ~D[1943-05-11]},
    ...>   %{name: "Lovelace", birthday: ~D[1815-12-10]},
    ...>   %{name: "Turing", birthday: ~D[1912-06-23]}
    ...> ]
    iex> Enum.sort_by(users, &(&1.birthday), Date)
    [
      %{name: "Lovelace", birthday: ~D[1815-12-10]},
      %{name: "Turing", birthday: ~D[1912-06-23]},
      %{name: "Ellis", birthday: ~D[1943-05-11]}
    ]
    iex> Enum.sort_by(users, &(&1.birthday), {:desc, Date})
    [
      %{name: "Ellis", birthday: ~D[1943-05-11]},
      %{name: "Turing", birthday: ~D[1912-06-23]},
      %{name: "Lovelace", birthday: ~D[1815-12-10]}
    ]

#### Performance characteristics

As detailed in the initial section, `sort_by/3` calculates the comparison
value for each element in the enumerable once instead of once for each
element in each comparison. This implies `sort_by/3` must do an initial
pass on the data to compute those values.

However, if those values are cheap to compute, for example, you have
already extracted the field you want to sort by into a tuple, then those
extra passes become overhead. In such cases, consider using `List.keysort/3`
instead.

Let's see an example. Imagine you have a list of products and you have a
list of IDs. You want to keep all products that are in the given IDs and
return their names sorted by their price. You could write it like this:

    for(
      product <- products,
      product.id in ids,
      do: product
    )
    |> Enum.sort_by(& &1.price)
    |> Enum.map(& &1.name)

However, you could also write it like this:

    for(
      product <- products,
      product.id in ids,
      do: {product.name, product.price}
    )
    |> List.keysort(1)
    |> Enum.map(&elem(&1, 0))

Using `List.keysort/3` will be a better choice for performance sensitive
code as it avoids additional traversals.

### split(enumerable, count)

```elixir
@spec split(t(), integer()) :: {list(), list()}
```

Splits the `enumerable` into two enumerables, leaving `count`
elements in the first one.

If `count` is a negative number, it starts counting from the
back to the beginning of the `enumerable`.

Be aware that a negative `count` implies the `enumerable`
will be enumerated twice: once to calculate the position, and
a second time to do the actual splitting.

#### Examples

    iex> Enum.split([1, 2, 3], 2)
    {[1, 2], [3]}
    
    iex> Enum.split([1, 2, 3], 10)
    {[1, 2, 3], []}
    
    iex> Enum.split([1, 2, 3], 0)
    {[], [1, 2, 3]}
    
    iex> Enum.split([1, 2, 3], -1)
    {[1, 2], [3]}
    
    iex> Enum.split([1, 2, 3], -5)
    {[], [1, 2, 3]}

### split_while(enumerable, fun)

```elixir
@spec split_while(t(), (element() -&gt; as_boolean(term()))) :: {list(), list()}
```

Splits enumerable in two at the position of the element for which
`fun` returns a falsy value (`false` or `nil`) for the first time.

It returns a two-element tuple with two lists of elements.
The element that triggered the split is part of the second list.

#### Examples

    iex> Enum.split_while([1, 2, 3, 4], fn x -> x < 3 end)
    {[1, 2], [3, 4]}
    
    iex> Enum.split_while([1, 2, 3, 4], fn x -> x < 0 end)
    {[], [1, 2, 3, 4]}
    
    iex> Enum.split_while([1, 2, 3, 4], fn x -> x > 0 end)
    {[1, 2, 3, 4], []}

### split_with(enumerable, fun)
*(since 1.4.0)* 
```elixir
@spec split_with(t(), (element() -&gt; as_boolean(term()))) :: {list(), list()}
```

Splits the `enumerable` in two lists according to the given function `fun`.

Splits the given `enumerable` in two lists by calling `fun` with each element
in the `enumerable` as its only argument. Returns a tuple with the first list
containing all the elements in `enumerable` for which applying `fun` returned
a truthy value, and a second list with all the elements for which applying
`fun` returned a falsy value (`false` or `nil`).

The elements in both the returned lists are in the same relative order as they
were in the original enumerable (if such enumerable was ordered, like a
list). See the examples below.

#### Examples

    iex> Enum.split_with([5, 4, 3, 2, 1, 0], fn x -> rem(x, 2) == 0 end)
    {[4, 2, 0], [5, 3, 1]}
    
    iex> Enum.split_with([a: 1, b: -2, c: 1, d: -3], fn {_k, v} -> v < 0 end)
    {[b: -2, d: -3], [a: 1, c: 1]}
    
    iex> Enum.split_with([a: 1, b: -2, c: 1, d: -3], fn {_k, v} -> v > 50 end)
    {[], [a: 1, b: -2, c: 1, d: -3]}
    
    iex> Enum.split_with([], fn {_k, v} -> v > 50 end)
    {[], []}

### sum(enumerable)

```elixir
@spec sum(t()) :: number()
```

Returns the sum of all elements.

Raises `ArithmeticError` if `enumerable` contains a non-numeric value.

If you need to apply a transformation first, consider using `Enum.sum_by/2` instead.

#### Examples

    iex> Enum.sum([1, 2, 3])
    6
    
    iex> Enum.sum(1..10)
    55
    
    iex> Enum.sum(1..10//2)
    25

### sum_by(enumerable, mapper)
*(since 1.18.0)* 
```elixir
@spec sum_by(t(), (element() -&gt; number())) :: number()
```

Maps and sums the given `enumerable` in one pass.

Raises `ArithmeticError` if `mapper` returns a non-numeric value.

#### Examples

    iex> Enum.sum_by([%{count: 1}, %{count: 2}, %{count: 3}], fn x -> x.count end)
    6
    
    iex> Enum.sum_by(1..3, fn x -> x ** 2 end)
    14
    
    iex> Enum.sum_by([], fn x -> x.count end)
    0

Filtering can be achieved by returning `0` to ignore elements:

    iex> Enum.sum_by([1, -2, 3], fn x -> if x > 0, do: x, else: 0 end)
    4

### take(enumerable, amount)

```elixir
@spec take(t(), integer()) :: list()
```

Takes an `amount` of elements from the beginning or the end of the `enumerable`.

If a positive `amount` is given, it takes the `amount` elements from the
beginning of the `enumerable`.

If a negative `amount` is given, the `amount` of elements will be taken from the end.
The `enumerable` will be enumerated once to retrieve the proper index and
the remaining calculation is performed from the end.

If amount is `0`, it returns `[]`.

#### Examples

    iex> Enum.take([1, 2, 3], 2)
    [1, 2]
    
    iex> Enum.take([1, 2, 3], 10)
    [1, 2, 3]
    
    iex> Enum.take([1, 2, 3], 0)
    []
    
    iex> Enum.take([1, 2, 3], -1)
    [3]

### take_every(enumerable, nth)

```elixir
@spec take_every(t(), non_neg_integer()) :: list()
```

Returns a list of every `nth` element in the `enumerable`,
starting with the first element.

The first element is always included, unless `nth` is 0.

The second argument specifying every `nth` element must be a non-negative
integer.

#### Examples

    iex> Enum.take_every(1..10, 2)
    [1, 3, 5, 7, 9]
    
    iex> Enum.take_every(1..10, 0)
    []
    
    iex> Enum.take_every([1, 2, 3], 1)
    [1, 2, 3]

### take_random(enumerable, count)

```elixir
@spec take_random(t(), non_neg_integer()) :: list()
```

Takes `count` random elements from `enumerable`.

Note that this function will traverse the whole `enumerable` to
get the random sublist.

See `random/1` for notes on implementation and random seed.

#### Examples

    # Although not necessary, let's seed the random algorithm
    iex> :rand.seed(:exsss, {1, 2, 3})
    iex> Enum.take_random(1..10, 2)
    [6, 1]
    iex> Enum.take_random(?a..?z, 5)
    ~c"bkzmt"

### take_while(enumerable, fun)

```elixir
@spec take_while(t(), (element() -&gt; as_boolean(term()))) :: list()
```

Takes the elements from the beginning of the `enumerable` while `fun` returns
a truthy value.

#### Examples

    iex> Enum.take_while([1, 2, 3], fn x -> x < 3 end)
    [1, 2]

### to_list(enumerable)

```elixir
@spec to_list(t()) :: [element()]
```

Converts `enumerable` to a list.

#### Examples

    iex> Enum.to_list(1..3)
    [1, 2, 3]

### uniq(enumerable)

```elixir
@spec uniq(t()) :: list()
```

Enumerates the `enumerable`, removing all duplicate elements.

#### Examples

    iex> Enum.uniq([1, 2, 3, 3, 2, 1])
    [1, 2, 3]

### uniq_by(enumerable, fun)

```elixir
@spec uniq_by(t(), (element() -&gt; term())) :: list()
```

Enumerates the `enumerable`, by removing the elements for which
function `fun` returned duplicate elements.

The function `fun` maps every element to a term. Two elements are
considered duplicates if the return value of `fun` is equal for
both of them.

The first occurrence of each element is kept.

#### Example

    iex> Enum.uniq_by([{1, :x}, {2, :y}, {1, :z}], fn {x, _} -> x end)
    [{1, :x}, {2, :y}]
    
    iex> Enum.uniq_by([a: {:tea, 2}, b: {:tea, 2}, c: {:coffee, 1}], fn {_, y} -> y end)
    [a: {:tea, 2}, c: {:coffee, 1}]

### unzip(list)

```elixir
@spec unzip(t()) :: {[element()], [element()]}
```

Opposite of `zip/2`. Extracts two-element tuples from the
given `enumerable` and groups them together.

It takes an `enumerable` with elements being two-element tuples and returns
a tuple with two lists, each of which is formed by the first and
second element of each tuple, respectively.

This function fails unless `enumerable` is or can be converted into a
list of tuples with *exactly* two elements in each tuple.

#### Examples

    iex> Enum.unzip([{:a, 1}, {:b, 2}, {:c, 3}])
    {[:a, :b, :c], [1, 2, 3]}

### with_index(enumerable, fun_or_offset \\ 0)

```elixir
@spec with_index(t(), integer()) :: [{term(), integer()}]
@spec with_index(t(), (element(), index() -&gt; value)) :: [value] when value: any()
```

Returns the `enumerable` with each element wrapped in a tuple
alongside its index or according to a given function.

If an integer offset is given as `fun_or_offset`, it will index from the given
offset instead of from zero.

If a function is given as `fun_or_offset`, it will index by invoking the function
for each element and index (zero-based) of the enumerable.

#### Examples

    iex> Enum.with_index([:a, :b, :c])
    [a: 0, b: 1, c: 2]
    
    iex> Enum.with_index([:a, :b, :c], 3)
    [a: 3, b: 4, c: 5]
    
    iex> Enum.with_index([:a, :b, :c], fn element, index -> {index, element} end)
    [{0, :a}, {1, :b}, {2, :c}]

### zip(enumerables)
*(since 1.4.0)* 
```elixir
@spec zip(enumerables) :: [tuple()] when enumerables: [t()] | t()
```

Zips corresponding elements from a finite collection of enumerables
into a list of tuples.

The zipping finishes as soon as any enumerable in the given collection completes.

#### Examples

    iex> Enum.zip([[1, 2, 3], [:a, :b, :c], ["foo", "bar", "baz"]])
    [{1, :a, "foo"}, {2, :b, "bar"}, {3, :c, "baz"}]
    
    iex> Enum.zip([[1, 2, 3, 4, 5], [:a, :b, :c]])
    [{1, :a}, {2, :b}, {3, :c}]

### zip(enumerable1, enumerable2)

```elixir
@spec zip(t(), t()) :: [{any(), any()}]
```

Zips corresponding elements from two enumerables into a list
of tuples.

Because a list of two-element tuples with atoms as the first
tuple element is a keyword list (`Keyword`), zipping a first list
of atoms with a second list of any kind creates a keyword list.

The zipping finishes as soon as either enumerable completes.

#### Examples

    iex> Enum.zip([1, 2, 3], [:a, :b, :c])
    [{1, :a}, {2, :b}, {3, :c}]
    
    iex> Enum.zip([:a, :b, :c], [1, 2, 3])
    [a: 1, b: 2, c: 3]
    
    iex> Enum.zip([1, 2, 3, 4, 5], [:a, :b, :c])
    [{1, :a}, {2, :b}, {3, :c}]

### zip_reduce(enums, acc, reducer)
*(since 1.12.0)* 
```elixir
@spec zip_reduce(t(), acc, ([term()], acc -&gt; acc)) :: acc when acc: term()
```

Reduces over all of the given enumerables, halting as soon as any enumerable is
empty.

The reducer will receive 2 args: a list of elements (one from each enum) and the
accumulator.

In practice, the behavior provided by this function can be achieved with:

    Enum.reduce(Stream.zip(enums), acc, reducer)

But `zip_reduce/3` exists for convenience purposes.

#### Examples

    iex> enums = [[1, 1], [2, 2], [3, 3]]
    ...>  Enum.zip_reduce(enums, [], fn elements, acc ->
    ...>    [List.to_tuple(elements) | acc]
    ...> end)
    [{1, 2, 3}, {1, 2, 3}]
    
    iex> enums = [[1, 2], [a: 3, b: 4], [5, 6]]
    ...> Enum.zip_reduce(enums, [], fn elements, acc ->
    ...>   [List.to_tuple(elements) | acc]
    ...> end)
    [{2, {:b, 4}, 6}, {1, {:a, 3}, 5}]

### zip_reduce(left, right, acc, reducer)
*(since 1.12.0)* 
```elixir
@spec zip_reduce(t(), t(), acc, (enum1_elem :: term(), enum2_elem :: term(), acc -&gt;
                             acc)) :: acc
when acc: term()
```

Reduces over two enumerables halting as soon as either enumerable is empty.

In practice, the behavior provided by this function can be achieved with:

    Enum.reduce(Stream.zip(left, right), acc, reducer)

But `zip_reduce/4` exists for convenience purposes.

#### Examples

    iex> Enum.zip_reduce([1, 2], [3, 4], 0, fn x, y, acc -> x + y + acc end)
    10
    
    iex> Enum.zip_reduce([1, 2], [3, 4], [], fn x, y, acc -> [x + y | acc] end)
    [6, 4]

### zip_with(enumerables, zip_fun)
*(since 1.12.0)* 
```elixir
@spec zip_with(t(), ([term()] -&gt; term())) :: [term()]
```

Zips corresponding elements from a finite collection of enumerables
into list, transforming them with the `zip_fun` function as it goes.

The first element from each of the enums in `enumerables` will be put
into a list which is then passed to the one-arity `zip_fun` function.
Then, the second elements from each of the enums are put into a list
and passed to `zip_fun`, and so on until any one of the enums in
`enumerables` runs out of elements.

Returns a list with all the results of calling `zip_fun`.

#### Examples

    iex> Enum.zip_with([[1, 2], [3, 4], [5, 6]], fn [x, y, z] -> x + y + z end)
    [9, 12]
    
    iex> Enum.zip_with([[1, 2], [3, 4]], fn [x, y] -> x + y end)
    [4, 6]

### zip_with(enumerable1, enumerable2, zip_fun)
*(since 1.12.0)* 
```elixir
@spec zip_with(t(), t(), (enum1_elem :: term(), enum2_elem :: term() -&gt; term())) :: [
  term()
]
```

Zips corresponding elements from two enumerables into a list, transforming them with
the `zip_fun` function as it goes.

The corresponding elements from each collection are passed to the provided two-arity `zip_fun`
function in turn. Returns a list that contains the result of calling `zip_fun` for each pair of
elements.

The zipping finishes as soon as either enumerable runs out of elements.

#### Zipping Maps

It's important to remember that zipping inherently relies on order.
If you zip two lists you get the element at the index from each list in turn.
If we zip two maps together it's tempting to think that you will get the given
key in the left map and the matching key in the right map, but there is no such
guarantee because map keys are not ordered\! Consider the following:

    left =  %{:a => 1, 1 => 3}
    right = %{:a => 1, :b => :c}
    Enum.zip(left, right)
    # [{{1, 3}, {:a, 1}}, {{:a, 1}, {:b, :c}}]

As you can see `:a` does not get paired with `:a`. If this is what you want,
you should use `Map.merge/3`.

#### Examples

    iex> Enum.zip_with([1, 2], [3, 4], fn x, y -> x + y end)
    [4, 6]
    
    iex> Enum.zip_with([1, 2], [3, 4, 5, 6], fn x, y -> x + y end)
    [4, 6]
    
    iex> Enum.zip_with([1, 2, 5, 6], [3, 4], fn x, y -> x + y end)
    [4, 6]



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
