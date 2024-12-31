# Map 
(Elixir v1.18.0-dev)

Maps are the "go to" key-value data structure in Elixir.

Maps can be created with the `%{}` syntax, and key-value pairs can be
expressed as `key => value`:

    iex> %{}
    %{}
    iex> %{"one" => :two, 3 => "four"}
    %{3 => "four", "one" => :two}

Key-value pairs in a map do not follow any order (that's why the printed map
in the example above has a different order than the map that was created).

Maps do not impose any restriction on the key type: anything can be a key in a
map. As a key-value structure, maps do not allow duplicate keys. Keys are
compared using the exact-equality operator (`===/2`). If colliding keys are defined
in a map literal, the last one prevails.

When the key in a key-value pair is an atom, the `key: value` shorthand syntax
can be used (as in many other special forms):

    iex> %{a: 1, b: 2}
    %{a: 1, b: 2}

If you want to mix the shorthand syntax with `=>`, the shorthand syntax must come
at the end:

    iex> %{"hello" => "world", a: 1, b: 2}
    %{:a => 1, :b => 2, "hello" => "world"}

Keys in maps can be accessed through some of the functions in this module
(such as `Map.get/3` or `Map.fetch/2`) or through the `map[]` syntax provided
by the `Access` module:

    iex> map = %{a: 1, b: 2}
    iex> Map.fetch(map, :a)
    {:ok, 1}
    iex> map[:b]
    2
    iex> map["non_existing_key"]
    nil

To access atom keys, one may also use the `map.key` notation. Note that `map.key`
will raise a `KeyError` if the `map` doesn't contain the key `:key`, compared to
`map[:key]`, that would return `nil`.

    map = %{foo: "bar", baz: "bong"}
    map.foo
    #=> "bar"
    map.non_existing_key
    ** (KeyError) key :non_existing_key not found in: %{baz: "bong", foo: "bar"}

> #### Avoid parentheses {: .warning}
> 
> Do not add parentheses when accessing fields, such as in `data.key()`.
> If parentheses are used, Elixir will expect `data` to be an atom representing
> a module and attempt to call the *function* `key/0` in it.

The two syntaxes for accessing keys reveal the dual nature of maps. The `map[key]`
syntax is used for dynamically created maps that may have any key, of any type.
`map.key` is used with maps that hold a predetermined set of atoms keys, which are
expected to always be present. Structs, defined via `defstruct/1`, are one example
of such "static maps", where the keys can also be checked during compile time.

Maps can be pattern matched on. When a map is on the left-hand side of a
pattern match, it will match if the map on the right-hand side contains the
keys on the left-hand side and their values match the ones on the left-hand
side. This means that an empty map matches every map.

    iex> %{} = %{foo: "bar"}
    %{foo: "bar"}
    iex> %{a: a} = %{:a => 1, "b" => 2, [:c, :e, :e] => 3}
    iex> a
    1

But this will raise a `MatchError` exception:

    %{:c => 3} = %{:a => 1, 2 => :b}

Variables can be used as map keys both when writing map literals as well as
when matching:

    iex> n = 1
    1
    iex> %{n => :one}
    %{1 => :one}
    iex> %{^n => :one} = %{1 => :one, 2 => :two, 3 => :three}
    %{1 => :one, 2 => :two, 3 => :three}

Maps also support a specific update syntax to update the value stored under
*existing* keys. You can update using the atom keys syntax:

    iex> map = %{one: 1, two: 2}
    iex> %{map | one: "one"}
    %{one: "one", two: 2}

Or any other key:

    iex> other_map = %{"three" => 3, "four" => 4}
    iex> %{other_map | "three" => "three"}
    %{"four" => 4, "three" => "three"}

When a key that does not exist in the map is updated a `KeyError` exception will be raised:

    %{map | three: 3}

The functions in this module that need to find a specific key work in logarithmic time.
This means that the time it takes to find keys grows as the map grows, but it's not
directly proportional to the map size. In comparison to finding an element in a list,
it performs better because lists have a linear time complexity. Some functions,
such as `keys/1` and `values/1`, run in linear time because they need to get to every
element in the map.

Maps also implement the `Enumerable` protocol, so many functions to work with maps
are found in the `Enum` module. Additionally, the following functions for maps are
found in `Kernel`:

- `map_size/1`

## Types

### key()

```elixir
@type key() :: any()
```



### value()

```elixir
@type value() :: any()
```



## Functions

### delete(map, key)

```elixir
@spec delete(map(), key()) :: map()
```

Deletes the entry in `map` for a specific `key`.

If the `key` does not exist, returns `map` unchanged.

Inlined by the compiler.

#### Examples

    iex> Map.delete(%{a: 1, b: 2}, :a)
    %{b: 2}
    iex> Map.delete(%{b: 2}, :a)
    %{b: 2}

### drop(map, keys)

```elixir
@spec drop(map(), [key()]) :: map()
```

Drops the given `keys` from `map`.

If `keys` contains keys that are not in `map`, they're simply ignored.

#### Examples

    iex> Map.drop(%{a: 1, b: 2, c: 3}, [:b, :d])
    %{a: 1, c: 3}

### equal?(map1, map2)

```elixir
@spec equal?(map(), map()) :: boolean()
```

Checks if two maps are equal.

Two maps are considered to be equal if they contain
the same keys and those keys contain the same values.

Note this function exists for completeness so the `Map`
and `Keyword` modules provide similar APIs. In practice,
developers often compare maps using `==/2` or `===/2`
directly.

#### Examples

    iex> Map.equal?(%{a: 1, b: 2}, %{b: 2, a: 1})
    true
    iex> Map.equal?(%{a: 1, b: 2}, %{b: 1, a: 2})
    false

Comparison between keys and values is done with `===/3`,
which means integers are not equivalent to floats:

    iex> Map.equal?(%{a: 1.0}, %{a: 1})
    false

### fetch(map, key)

```elixir
@spec fetch(map(), key()) :: {:ok, value()} | :error
```

Fetches the value for a specific `key` in the given `map`.

If `map` contains the given `key` then its value is returned in the shape of `{:ok, value}`.
If `map` doesn't contain `key`, `:error` is returned.

Inlined by the compiler.

#### Examples

    iex> Map.fetch(%{a: 1}, :a)
    {:ok, 1}
    iex> Map.fetch(%{a: 1}, :b)
    :error

### fetch!(map, key)

```elixir
@spec fetch!(map(), key()) :: value()
```

Fetches the value for a specific `key` in the given `map`, erroring out if
`map` doesn't contain `key`.

If `map` contains `key`, the corresponding value is returned. If
`map` doesn't contain `key`, a `KeyError` exception is raised.

Inlined by the compiler.

#### Examples

    iex> Map.fetch!(%{a: 1}, :a)
    1

### filter(map, fun)
*(since 1.13.0)* 
```elixir
@spec filter(map(), ({key(), value()} -&gt; as_boolean(term()))) :: map()
```

Returns a map containing only those pairs from `map`
for which `fun` returns a truthy value.

`fun` receives the key and value of each of the
elements in the map as a key-value pair.

See also `reject/2` which discards all elements where the
function returns a truthy value.

> #### Performance considerations {: .tip}
> 
> If you find yourself doing multiple calls to `Map.filter/2`
> and `Map.reject/2` in a pipeline, it is likely more efficient
> to use `Enum.map/2` and `Enum.filter/2` instead and convert to
> a map at the end using `Map.new/1`.

#### Examples

    iex> Map.filter(%{one: 1, two: 2, three: 3}, fn {_key, val} -> rem(val, 2) == 1 end)
    %{one: 1, three: 3}

### from_keys(keys, value)
*(since 1.14.0)* 
```elixir
@spec from_keys([key()], value()) :: map()
```

Builds a map from the given `keys` and the fixed `value`.

Inlined by the compiler.

#### Examples

    iex> Map.from_keys([1, 2, 3], :number)
    %{1 => :number, 2 => :number, 3 => :number}

### from_struct(struct)

```elixir
@spec from_struct(atom() | struct()) :: map()
```

Converts a `struct` to map.

It accepts a struct and simply removes the `__struct__` field
from the given struct.

#### Example

    defmodule User do
      defstruct [:name]
    end
    
    Map.from_struct(%User{name: "john"})
    #=> %{name: "john"}

### get(map, key, default \\ nil)

```elixir
@spec get(map(), key(), value()) :: value()
```

Gets the value for a specific `key` in `map`.

If `key` is present in `map` then its value `value` is
returned. Otherwise, `default` is returned.

If `default` is not provided, `nil` is used.

#### Examples

    iex> Map.get(%{}, :a)
    nil
    iex> Map.get(%{a: 1}, :a)
    1
    iex> Map.get(%{a: 1}, :b)
    nil
    iex> Map.get(%{a: 1}, :b, 3)
    3
    iex> Map.get(%{a: nil}, :a, 1)
    nil

### get_and_update(map, key, fun)

```elixir
@spec get_and_update(map(), key(), (value() | nil -&gt;
                                {current_value, new_value :: value()} | :pop)) ::
  {current_value, new_map :: map()}
when current_value: value()
```

Gets the value from `key` and updates it, all in one pass.

`fun` is called with the current value under `key` in `map` (or `nil` if `key`
is not present in `map`) and must return a two-element tuple: the current value
(the retrieved value, which can be operated on before being returned) and the
new value to be stored under `key` in the resulting new map. `fun` may also
return `:pop`, which means the current value shall be removed from `map` and
returned (making this function behave like `Map.pop(map, key)`).

The returned value is a two-element tuple with the current value returned by
`fun` and a new map with the updated value under `key`.

#### Examples

    iex> Map.get_and_update(%{a: 1}, :a, fn current_value ->
    ...>   {current_value, "new value!"}
    ...> end)
    {1, %{a: "new value!"}}
    
    iex> Map.get_and_update(%{a: 1}, :b, fn current_value ->
    ...>   {current_value, "new value!"}
    ...> end)
    {nil, %{a: 1, b: "new value!"}}
    
    iex> Map.get_and_update(%{a: 1}, :a, fn _ -> :pop end)
    {1, %{}}
    
    iex> Map.get_and_update(%{a: 1}, :b, fn _ -> :pop end)
    {nil, %{a: 1}}

### get_and_update!(map, key, fun)

```elixir
@spec get_and_update!(map(), key(), (value() -&gt;
                                 {current_value, new_value :: value()} | :pop)) ::
  {current_value, map()}
when current_value: value()
```

Gets the value from `key` and updates it, all in one pass. Raises if there is no `key`.

Behaves exactly like `get_and_update/3`, but raises a `KeyError` exception if
`key` is not present in `map`.

#### Examples

    iex> Map.get_and_update!(%{a: 1}, :a, fn current_value ->
    ...>   {current_value, "new value!"}
    ...> end)
    {1, %{a: "new value!"}}
    
    iex> Map.get_and_update!(%{a: 1}, :b, fn current_value ->
    ...>   {current_value, "new value!"}
    ...> end)
    ** (KeyError) key :b not found in: %{a: 1}
    
    iex> Map.get_and_update!(%{a: 1}, :a, fn _ ->
    ...>   :pop
    ...> end)
    {1, %{}}

### get_lazy(map, key, fun)

```elixir
@spec get_lazy(map(), key(), (-&gt; value())) :: value()
```

Gets the value for a specific `key` in `map`.

If `key` is present in `map` then its value `value` is
returned. Otherwise, `fun` is evaluated and its result is returned.

This is useful if the default value is very expensive to calculate or
generally difficult to setup and teardown again.

#### Examples

    iex> map = %{a: 1}
    iex> fun = fn ->
    ...>   # some expensive operation here
    ...>   13
    ...> end
    iex> Map.get_lazy(map, :a, fun)
    1
    iex> Map.get_lazy(map, :b, fun)
    13

### has_key?(map, key)

```elixir
@spec has_key?(map(), key()) :: boolean()
```

Returns whether the given `key` exists in the given `map`.

Inlined by the compiler.

#### Examples

    iex> Map.has_key?(%{a: 1}, :a)
    true
    iex> Map.has_key?(%{a: 1}, :b)
    false

### intersect(map1, map2)
*(since 1.15.0)* 
```elixir
@spec intersect(map(), map()) :: map()
```

Intersects two maps, returning a map with the common keys.

The values in the returned map are the values of the intersected keys in `map2`.

Inlined by the compiler.

#### Examples

    iex> Map.intersect(%{a: 1, b: 2}, %{b: "b", c: "c"})
    %{b: "b"}

### intersect(map1, map2, fun)
*(since 1.15.0)* 
```elixir
@spec intersect(map(), map(), (key(), value(), value() -&gt; value())) :: map()
```

Intersects two maps, returning a map with the common keys and resolving conflicts through a function.

The given function will be invoked when there are duplicate keys; its
arguments are `key` (the duplicate key), `value1` (the value of `key` in
`map1`), and `value2` (the value of `key` in `map2`). The value returned by
`fun` is used as the value under `key` in the resulting map.

#### Examples

    iex> Map.intersect(%{a: 1, b: 2}, %{b: 2, c: 3}, fn _k, v1, v2 ->
    ...>   v1 + v2
    ...> end)
    %{b: 4}

### keys(map)

```elixir
@spec keys(map()) :: [key()]
```

Returns all keys from `map`.

Inlined by the compiler.

#### Examples

    Map.keys(%{a: 1, b: 2})
    [:a, :b]

### merge(map1, map2)

```elixir
@spec merge(map(), map()) :: map()
```

Merges two maps into one.

All keys in `map2` will be added to `map1`, overriding any existing one
(i.e., the keys in `map2` "have precedence" over the ones in `map1`).

If you have a struct and you would like to merge a set of keys into the
struct, do not use this function, as it would merge all keys on the right
side into the struct, even if the key is not part of the struct. Instead,
use `struct/2`.

Inlined by the compiler.

#### Examples

    iex> Map.merge(%{a: 1, b: 2}, %{a: 3, d: 4})
    %{a: 3, b: 2, d: 4}

### merge(map1, map2, fun)

```elixir
@spec merge(map(), map(), (key(), value(), value() -&gt; value())) :: map()
```

Merges two maps into one, resolving conflicts through the given `fun`.

All keys in `map2` will be added to `map1`. The given function will be invoked
when there are duplicate keys; its arguments are `key` (the duplicate key),
`value1` (the value of `key` in `map1`), and `value2` (the value of `key` in
`map2`). The value returned by `fun` is used as the value under `key` in
the resulting map.

#### Examples

    iex> Map.merge(%{a: 1, b: 2}, %{a: 3, d: 4}, fn _k, v1, v2 ->
    ...>   v1 + v2
    ...> end)
    %{a: 4, b: 2, d: 4}

### new()

```elixir
@spec new() :: map()
```

Returns a new empty map.

#### Examples

    iex> Map.new()
    %{}

### new(enumerable)

```elixir
@spec new(Enumerable.t()) :: map()
```

Creates a map from an `enumerable`.

Duplicated keys are removed; the latest one prevails.

#### Examples

    iex> Map.new([{:b, 1}, {:a, 2}])
    %{a: 2, b: 1}
    iex> Map.new(a: 1, a: 2, a: 3)
    %{a: 3}

### new(enumerable, transform)

```elixir
@spec new(Enumerable.t(), (term() -&gt; {key(), value()})) :: map()
```

Creates a map from an `enumerable` via the given transformation function.

Duplicated keys are removed; the latest one prevails.

#### Examples

    iex> Map.new([:a, :b], fn x -> {x, x} end)
    %{a: :a, b: :b}
    
    iex> Map.new(%{a: 2, b: 3, c: 4}, fn {key, val} -> {key, val * 2} end)
    %{a: 4, b: 6, c: 8}

### pop(map, key, default \\ nil)

```elixir
@spec pop(map(), key(), default) :: {value(), updated_map :: map()} | {default, map()}
when default: value()
```

Removes the value associated with `key` in `map` and returns the value and the updated map.

If `key` is present in `map`, it returns `{value, updated_map}` where `value` is the value of
the key and `updated_map` is the result of removing `key` from `map`. If `key`
is not present in `map`, `{default, map}` is returned.

#### Examples

    iex> Map.pop(%{a: 1}, :a)
    {1, %{}}
    iex> Map.pop(%{a: 1}, :b)
    {nil, %{a: 1}}
    iex> Map.pop(%{a: 1}, :b, 3)
    {3, %{a: 1}}

### pop!(map, key)
*(since 1.10.0)* 
```elixir
@spec pop!(map(), key()) :: {value(), updated_map :: map()}
```

Removes and returns the value associated with `key` in `map` alongside
the updated map, or raises if `key` is not present.

Behaves the same as `pop/3` but raises a `KeyError` exception if `key` is not present in `map`.

#### Examples

    iex> Map.pop!(%{a: 1}, :a)
    {1, %{}}
    iex> Map.pop!(%{a: 1, b: 2}, :a)
    {1, %{b: 2}}
    iex> Map.pop!(%{a: 1}, :b)
    ** (KeyError) key :b not found in: %{a: 1}

### pop_lazy(map, key, fun)

```elixir
@spec pop_lazy(map(), key(), (-&gt; value())) :: {value(), map()}
```

Lazily returns and removes the value associated with `key` in `map`.

If `key` is present in `map`, it returns `{value, new_map}` where `value` is the value of
the key and `new_map` is the result of removing `key` from `map`. If `key`
is not present in `map`, `{fun_result, map}` is returned, where `fun_result`
is the result of applying `fun`.

This is useful if the default value is very expensive to calculate or
generally difficult to setup and teardown again.

#### Examples

    iex> map = %{a: 1}
    iex> fun = fn ->
    ...>   # some expensive operation here
    ...>   13
    ...> end
    iex> Map.pop_lazy(map, :a, fun)
    {1, %{}}
    iex> Map.pop_lazy(map, :b, fun)
    {13, %{a: 1}}

### put(map, key, value)

```elixir
@spec put(map(), key(), value()) :: map()
```

Puts the given `value` under `key` in `map`.

Inlined by the compiler.

#### Examples

    iex> Map.put(%{a: 1}, :b, 2)
    %{a: 1, b: 2}
    iex> Map.put(%{a: 1, b: 2}, :a, 3)
    %{a: 3, b: 2}

### put_new(map, key, value)

```elixir
@spec put_new(map(), key(), value()) :: map()
```

Puts the given `value` under `key` unless the entry `key`
already exists in `map`.

#### Examples

    iex> Map.put_new(%{a: 1}, :b, 2)
    %{a: 1, b: 2}
    iex> Map.put_new(%{a: 1, b: 2}, :a, 3)
    %{a: 1, b: 2}

### put_new_lazy(map, key, fun)

```elixir
@spec put_new_lazy(map(), key(), (-&gt; value())) :: map()
```

Evaluates `fun` and puts the result under `key`
in `map` unless `key` is already present.

This function is useful in case you want to compute the value to put under
`key` only if `key` is not already present, as for example, when the value is expensive to
calculate or generally difficult to setup and teardown again.

#### Examples

    iex> map = %{a: 1}
    iex> fun = fn ->
    ...>   # some expensive operation here
    ...>   3
    ...> end
    iex> Map.put_new_lazy(map, :a, fun)
    %{a: 1}
    iex> Map.put_new_lazy(map, :b, fun)
    %{a: 1, b: 3}

### reject(map, fun)
*(since 1.13.0)* 
```elixir
@spec reject(map(), ({key(), value()} -&gt; as_boolean(term()))) :: map()
```

Returns map excluding the pairs from `map` for which `fun` returns
a truthy value.

See also `filter/2`.

#### Examples

    iex> Map.reject(%{one: 1, two: 2, three: 3}, fn {_key, val} -> rem(val, 2) == 1 end)
    %{two: 2}

### replace(map, key, value)
*(since 1.11.0)* 
```elixir
@spec replace(map(), key(), value()) :: map()
```

Puts a value under `key` only if the `key` already exists in `map`.

#### Examples

    iex> Map.replace(%{a: 1, b: 2}, :a, 3)
    %{a: 3, b: 2}
    
    iex> Map.replace(%{a: 1}, :b, 2)
    %{a: 1}

### replace!(map, key, value)
*(since 1.5.0)* 
```elixir
@spec replace!(map(), key(), value()) :: map()
```

Puts a value under `key` only if the `key` already exists in `map`.

If `key` is not present in `map`, a `KeyError` exception is raised.

Inlined by the compiler.

#### Examples

    iex> Map.replace!(%{a: 1, b: 2}, :a, 3)
    %{a: 3, b: 2}
    
    iex> Map.replace!(%{a: 1}, :b, 2)
    ** (KeyError) key :b not found in: %{a: 1}

### replace_lazy(map, key, fun)
*(since 1.14.0)* 
```elixir
@spec replace_lazy(map(), key(), (existing_value :: value() -&gt; new_value :: value())) ::
  map()
```

Replaces the value under `key` using the given function only if
`key` already exists in `map`.

In comparison to `replace/3`, this can be useful when it's expensive to calculate the value.

If `key` does not exist, the original map is returned unchanged.

#### Examples

    iex> Map.replace_lazy(%{a: 1, b: 2}, :a, fn v -> v * 4 end)
    %{a: 4, b: 2}
    
    iex> Map.replace_lazy(%{a: 1, b: 2}, :c, fn v -> v * 4 end)
    %{a: 1, b: 2}

### split(map, keys)

```elixir
@spec split(map(), [key()]) :: {map(), map()}
```

Takes all entries corresponding to the given `keys` in `map` and extracts
them into a separate map.

Returns a tuple with the new map and the old map with removed keys.

Keys for which there are no entries in `map` are ignored.

#### Examples

    iex> Map.split(%{a: 1, b: 2, c: 3}, [:a, :c, :e])
    {%{a: 1, c: 3}, %{b: 2}}

### split_with(map, fun)
*(since 1.15.0)* 
```elixir
@spec split_with(map(), ({key(), value()} -&gt; as_boolean(term()))) :: {map(), map()}
```

Splits the `map` into two maps according to the given function `fun`.

`fun` receives each `{key, value}` pair in the `map` as its only argument. Returns
a tuple with the first map containing all the elements in `map` for which
applying `fun` returned a truthy value, and a second map with all the elements
for which applying `fun` returned a falsy value (`false` or `nil`).

#### Examples

    iex> Map.split_with(%{a: 1, b: 2, c: 3, d: 4}, fn {_k, v} -> rem(v, 2) == 0 end)
    {%{b: 2, d: 4}, %{a: 1, c: 3}}
    
    iex> Map.split_with(%{a: 1, b: -2, c: 1, d: -3}, fn {k, _v} -> k in [:b, :d] end)
    {%{b: -2, d: -3}, %{a: 1, c: 1}}
    
    iex> Map.split_with(%{a: 1, b: -2, c: 1, d: -3}, fn {_k, v} -> v > 50 end)
    {%{}, %{a: 1, b: -2, c: 1, d: -3}}
    
    iex> Map.split_with(%{}, fn {_k, v} -> v > 50 end)
    {%{}, %{}}

### take(map, keys)

```elixir
@spec take(map(), [key()]) :: map()
```

Returns a new map with all the key-value pairs in `map` where the key
is in `keys`.

If `keys` contains keys that are not in `map`, they're simply ignored.

#### Examples

    iex> Map.take(%{a: 1, b: 2, c: 3}, [:a, :c, :e])
    %{a: 1, c: 3}

### to_list(map)

```elixir
@spec to_list(map()) :: [{term(), term()}]
```

Converts `map` to a list.

Each key-value pair in the map is converted to a two-element tuple `{key, value}` in the resulting list.

Inlined by the compiler.

#### Examples

    iex> Map.to_list(%{a: 1})
    [a: 1]
    iex> Map.to_list(%{1 => 2})
    [{1, 2}]

### update(map, key, default, fun)

```elixir
@spec update(map(), key(), default :: value(), (existing_value :: value() -&gt;
                                            new_value :: value())) ::
  map()
```

Updates the `key` in `map` with the given function.

If `key` is present in `map` then the existing value is passed to `fun` and its result is
used as the updated value of `key`. If `key` is
not present in `map`, `default` is inserted as the value of `key`. The default
value will not be passed through the update function.

#### Examples

    iex> Map.update(%{a: 1}, :a, 13, fn existing_value -> existing_value * 2 end)
    %{a: 2}
    iex> Map.update(%{a: 1}, :b, 11, fn existing_value -> existing_value * 2 end)
    %{a: 1, b: 11}

### update!(map, key, fun)

```elixir
@spec update!(map(), key(), (existing_value :: value() -&gt; new_value :: value())) ::
  map()
```

Updates `key` with the given function.

If `key` is present in `map` then the existing value is passed to `fun` and its result is
used as the updated value of `key`. If `key` is
not present in `map`, a `KeyError` exception is raised.

#### Examples

    iex> Map.update!(%{a: 1}, :a, &(&1 * 2))
    %{a: 2}
    
    iex> Map.update!(%{a: 1}, :b, &(&1 * 2))
    ** (KeyError) key :b not found in: %{a: 1}

### values(map)

```elixir
@spec values(map()) :: [value()]
```

Returns all values from `map`.

Inlined by the compiler.

#### Examples

    Map.values(%{a: 1, b: 2})
    [1, 2]



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
