# Keyword 
(Elixir v1.18.0-dev)

A keyword list is a list that consists exclusively of two-element tuples.

The first element of these tuples is known as the *key*, and it must be an atom.
The second element, known as the *value*, can be any term.

Keywords are mostly used to work with optional values. For a general introduction
to keywords and how they compare with maps, see our [Keyword and Maps](keywords-and-maps.md)
guide.

## Examples

For example, the following is a keyword list:

    [{:exit_on_close, true}, {:active, :once}, {:packet_size, 1024}]

Elixir provides a special and more concise syntax for keyword lists:

    [exit_on_close: true, active: :once, packet_size: 1024]

The two syntaxes return the exact same value.

A *key* can be any atom, consisting of Unicode letters, numbers,
an underscore or the `@` sign. If the *key* should have any other
characters, such as spaces, you can wrap it in quotes:

    iex> ["exit on close": true]
    ["exit on close": true]

Wrapping an atom in quotes does not make it a string. Keyword list
*keys* are always atoms. Quotes should only be used when necessary
or Elixir will issue a warning.

## Duplicate keys and ordering

A keyword may have duplicate keys so it is not strictly a key-value
data type. However, most of the functions in this module work on a
key-value structure and behave similar to the functions you would
find in the `Map` module. For example, `Keyword.get/3` will get the first
entry matching the given key, regardless if duplicate entries exist.
Similarly, `Keyword.put/3` and `Keyword.delete/2` ensure all duplicate
entries for a given key are removed when invoked. Note, however, that
keyword list operations need to traverse the whole list in order to find
keys, so these operations are slower than their map counterparts.

A handful of functions exist to handle duplicate keys, for example,
`get_values/2` returns all values for a given key and `delete_first/2`
deletes just the first entry of the existing ones.

Even though lists preserve the existing order, the functions in
`Keyword` do not guarantee any ordering. For example, if you invoke
`Keyword.put(opts, new_key, new_value)`, there is no guarantee for
where `new_key` will be added to (the front, the end or anywhere else).

Given ordering is not guaranteed, it is not recommended to pattern
match on keyword lists either. For example, a function such as:

    def my_function([some_key: value, another_key: another_value])

will match

    my_function([some_key: :foo, another_key: :bar])

but it won't match

    my_function([another_key: :bar, some_key: :foo])

Most of the functions in this module work in linear time. This means
that the time it takes to perform an operation grows at the same
rate as the length of the list.

## Call syntax

When keyword lists are passed as the last argument to a function,
the square brackets around the keyword list can be omitted. For
example, the keyword list syntax:

    String.split("1-0", "-", [trim: true, parts: 2])

can be written without the enclosing brackets whenever it is the last
argument of a function call:

    String.split("1-0", "-", trim: true, parts: 2)

Since tuples, lists and maps are treated similarly to function
arguments in Elixir syntax, this property is also available to them:

    iex> {1, 2, foo: :bar}
    {1, 2, [{:foo, :bar}]}
    
    iex> [1, 2, foo: :bar]
    [1, 2, {:foo, :bar}]
    
    iex> %{1 => 2, foo: :bar}
    %{1 => 2, :foo => :bar}


## Types

### default()
*(since 1.17.0)* 
```elixir
@type default() :: any()
```



### key()

```elixir
@type key() :: atom()
```



### t()

```elixir
@type t() :: [{key(), value()}]
```



### t(value)

```elixir
@type t(value) :: [{key(), value}]
```



### value()

```elixir
@type value() :: any()
```



## Functions

### delete(keywords, key)

```elixir
@spec delete(t(), key()) :: t()
```

Deletes the entries in the keyword list under a specific `key`.

If the `key` does not exist, it returns the keyword list unchanged.
Use `delete_first/2` to delete just the first entry in case of
duplicate keys.

#### Examples

    iex> Keyword.delete([a: 1, b: 2], :a)
    [b: 2]
    iex> Keyword.delete([a: 1, b: 2, a: 3], :a)
    [b: 2]
    iex> Keyword.delete([b: 2], :a)
    [b: 2]


### delete_first(keywords, key)

```elixir
@spec delete_first(t(), key()) :: t()
```

Deletes the first entry in the keyword list under a specific `key`.

If the `key` does not exist, it returns the keyword list unchanged.

#### Examples

    iex> Keyword.delete_first([a: 1, b: 2, a: 3], :a)
    [b: 2, a: 3]
    iex> Keyword.delete_first([b: 2], :a)
    [b: 2]


### drop(keywords, keys)

```elixir
@spec drop(t(), [key()]) :: t()
```

Drops the given `keys` from the keyword list.

Removes duplicate keys from the new keyword list.

#### Examples

    iex> Keyword.drop([a: 1, a: 2], [:a])
    []
    iex> Keyword.drop([a: 1, b: 2, c: 3], [:b, :d])
    [a: 1, c: 3]
    iex> Keyword.drop([a: 1, b: 2, b: 3, c: 3, a: 5], [:b, :d])
    [a: 1, c: 3, a: 5]


### equal?(left, right)

```elixir
@spec equal?(t(), t()) :: boolean()
```

Checks if two keywords are equal.

Considers two keywords to be equal if they contain
the same keys and those keys contain the same values.

#### Examples

    iex> Keyword.equal?([a: 1, b: 2], [b: 2, a: 1])
    true
    iex> Keyword.equal?([a: 1, b: 2], [b: 1, a: 2])
    false
    iex> Keyword.equal?([a: 1, b: 2, a: 3], [b: 2, a: 3, a: 1])
    true

Comparison between values is done with `===/3`,
which means integers are not equivalent to floats:

    iex> Keyword.equal?([a: 1.0], [a: 1])
    false


### fetch(keywords, key)

```elixir
@spec fetch(t(), key()) :: {:ok, value()} | :error
```

Fetches the value for a specific `key` and returns it in a tuple.

If the `key` does not exist, it returns `:error`.

#### Examples

    iex> Keyword.fetch([a: 1], :a)
    {:ok, 1}
    iex> Keyword.fetch([a: 1], :b)
    :error


### fetch!(keywords, key)

```elixir
@spec fetch!(t(), key()) :: value()
```

Fetches the value for specific `key`.

If the `key` does not exist, it raises a `KeyError`.

#### Examples

    iex> Keyword.fetch!([a: 1], :a)
    1
    iex> Keyword.fetch!([a: 1], :b)
    ** (KeyError) key :b not found in: [a: 1]


### filter(keywords, fun)
*(since 1.13.0)* 
```elixir
@spec filter(t(), ({key(), value()} -&gt; as_boolean(term()))) :: t()
```

Returns a keyword list containing only the entries from `keywords`
for which the function `fun` returns a truthy value.

See also `reject/2` which discards all entries where the function
returns a truthy value.

#### Examples

    iex> Keyword.filter([one: 1, two: 2, three: 3], fn {_key, val} -> rem(val, 2) == 1 end)
    [one: 1, three: 3]


### from_keys(keys, value)
*(since 1.14.0)* 
```elixir
@spec from_keys([key()], value()) :: t(value())
```

Builds a keyword from the given `keys` and the fixed `value`.

#### Examples

    iex> Keyword.from_keys([:foo, :bar, :baz], :atom)
    [foo: :atom, bar: :atom, baz: :atom]
    iex> Keyword.from_keys([], :atom)
    []


### get(keywords, key, default \\ nil)

```elixir
@spec get(t(), key(), default()) :: value() | default()
```

Gets the value under the given `key`.

Returns the default value if `key` does not exist
(`nil` if no default value is provided).

If duplicate entries exist, it returns the first one.
Use `get_values/2` to retrieve all entries.

#### Examples

    iex> Keyword.get([], :a)
    nil
    iex> Keyword.get([a: 1], :a)
    1
    iex> Keyword.get([a: 1], :b)
    nil
    iex> Keyword.get([a: 1], :b, 3)
    3

With duplicate keys:

    iex> Keyword.get([a: 1, a: 2], :a, 3)
    1
    iex> Keyword.get([a: 1, a: 2], :b, 3)
    3


### get_and_update(keywords, key, fun)

```elixir
@spec get_and_update(t(), key(), (value() | nil -&gt;
                              {current_value, new_value :: value()} | :pop)) ::
  {current_value, new_keywords :: t()}
when current_value: value()
```

Gets the value from `key` and updates it, all in one pass.

The `fun` argument receives the value of `key` (or `nil` if `key`
is not present) and must return a two-element tuple: the current value
(the retrieved value, which can be operated on before being returned)
and the new value to be stored under `key`. The `fun` may also
return `:pop`, implying the current value shall be removed from the
keyword list and returned.

Returns a tuple that contains the current value returned by
`fun` and a new keyword list with the updated value under `key`.

#### Examples

    iex> Keyword.get_and_update([a: 1], :a, fn current_value ->
    ...>   {current_value, "new value!"}
    ...> end)
    {1, [a: "new value!"]}
    
    iex> Keyword.get_and_update([a: 1], :b, fn current_value ->
    ...>   {current_value, "new value!"}
    ...> end)
    {nil, [b: "new value!", a: 1]}
    
    iex> Keyword.get_and_update([a: 2], :a, fn number ->
    ...>   {2 * number, 3 * number}
    ...> end)
    {4, [a: 6]}
    
    iex> Keyword.get_and_update([a: 1], :a, fn _ -> :pop end)
    {1, []}
    
    iex> Keyword.get_and_update([a: 1], :b, fn _ -> :pop end)
    {nil, [a: 1]}


### get_and_update!(keywords, key, fun)

```elixir
@spec get_and_update!(t(), key(), (value() -&gt;
                               {current_value, new_value :: value()} | :pop)) ::
  {current_value, new_keywords :: t()}
when current_value: value()
```

Gets the value under `key` and updates it. Raises if there is no `key`.

The `fun` argument receives the value under `key` and must return a
two-element tuple: the current value (the retrieved value, which can be
operated on before being returned) and the new value to be stored under
`key`.

Returns a tuple that contains the current value returned by
`fun` and a new keyword list with the updated value under `key`.

#### Examples

    iex> Keyword.get_and_update!([a: 1], :a, fn current_value ->
    ...>   {current_value, "new value!"}
    ...> end)
    {1, [a: "new value!"]}
    
    iex> Keyword.get_and_update!([a: 1], :b, fn current_value ->
    ...>   {current_value, "new value!"}
    ...> end)
    ** (KeyError) key :b not found in: [a: 1]
    
    iex> Keyword.get_and_update!([a: 1], :a, fn _ ->
    ...>   :pop
    ...> end)
    {1, []}


### get_lazy(keywords, key, fun)

```elixir
@spec get_lazy(t(), key(), (-&gt; value())) :: value()
```

Gets the value under the given `key`.

If `key` does not exist, lazily evaluates `fun` and returns its result.

This is useful if the default value is very expensive to calculate or
generally difficult to set up and tear down again.

If duplicate entries exist, it returns the first one.
Use `get_values/2` to retrieve all entries.

#### Examples

    iex> keyword = [a: 1]
    iex> fun = fn ->
    ...>   # some expensive operation here
    ...>   13
    ...> end
    iex> Keyword.get_lazy(keyword, :a, fun)
    1
    iex> Keyword.get_lazy(keyword, :b, fun)
    13


### get_values(keywords, key)

```elixir
@spec get_values(t(), key()) :: [value()]
```

Gets all values under a specific `key`.

#### Examples

    iex> Keyword.get_values([], :a)
    []
    iex> Keyword.get_values([a: 1], :a)
    [1]
    iex> Keyword.get_values([a: 1, a: 2], :a)
    [1, 2]


### has_key?(keywords, key)

```elixir
@spec has_key?(t(), key()) :: boolean()
```

Returns whether a given `key` exists in the given `keywords`.

#### Examples

    iex> Keyword.has_key?([a: 1], :a)
    true
    iex> Keyword.has_key?([a: 1], :b)
    false


### intersect(keyword1, keyword2, fun \\ fn _key, _v1, v2 -&gt; v2 end)
*(since 1.17.0)* 
```elixir
@spec intersect(keyword(), keyword(), (key(), value(), value() -&gt; value())) ::
  keyword()
```

Intersects two keyword lists, returning a keyword with the common keys.

By default, it returns the values of the intersected keys in `keyword2`.
The keys are returned in the order found in `keyword1`.

#### Examples

    iex> Keyword.intersect([a: 1, b: 2], [b: "b", c: "c"])
    [b: "b"]
    
    iex> Keyword.intersect([a: 1, b: 2], [b: 2, c: 3], fn _k, v1, v2 ->
    ...>   v1 + v2
    ...> end)
    [b: 4]


### keys(keywords)

```elixir
@spec keys(t()) :: [key()]
```

Returns all keys from the keyword list.

Keeps duplicate keys in the resulting list of keys.

#### Examples

    iex> Keyword.keys(a: 1, b: 2)
    [:a, :b]
    
    iex> Keyword.keys(a: 1, b: 2, a: 3)
    [:a, :b, :a]
    
    iex> Keyword.keys([{:a, 1}, {"b", 2}, {:c, 3}])
    ** (ArgumentError) expected a keyword list, but an entry in the list is not a two-element tuple with an atom as its first element, got: {"b", 2}


### keyword?(term)

```elixir
@spec keyword?(term()) :: boolean()
```

Returns `true` if `term` is a keyword list, otherwise `false`.

When `term` is a list it is traversed to the end.

#### Examples

    iex> Keyword.keyword?([])
    true
    iex> Keyword.keyword?(a: 1)
    true
    iex> Keyword.keyword?([{Foo, 1}])
    true
    iex> Keyword.keyword?([{}])
    false
    iex> Keyword.keyword?([:key])
    false
    iex> Keyword.keyword?(%{})
    false


### merge(keywords1, keywords2)

```elixir
@spec merge(t(), t()) :: t()
```

Merges two keyword lists into one.

Adds all keys, including duplicate keys, given in `keywords2`
to `keywords1`, overriding any existing ones.

There are no guarantees about the order of the keys in the returned keyword.

#### Examples

    iex> Keyword.merge([a: 1, b: 2], [a: 3, d: 4])
    [b: 2, a: 3, d: 4]
    
    iex> Keyword.merge([a: 1, b: 2], [a: 3, d: 4, a: 5])
    [b: 2, a: 3, d: 4, a: 5]
    
    iex> Keyword.merge([a: 1], [2, 3])
    ** (ArgumentError) expected a keyword list as the second argument, got: [2, 3]


### merge(keywords1, keywords2, fun)

```elixir
@spec merge(t(), t(), (key(), value(), value() -&gt; value())) :: t()
```

Merges two keyword lists into one.

Adds all keys, including duplicate keys, given in `keywords2`
to `keywords1`. Invokes the given function to solve conflicts.

If `keywords2` has duplicate keys, it invokes the given function
for each matching pair in `keywords1`.

There are no guarantees about the order of the keys in the returned keyword.

#### Examples

    iex> Keyword.merge([a: 1, b: 2], [a: 3, d: 4], fn _k, v1, v2 ->
    ...>   v1 + v2
    ...> end)
    [b: 2, a: 4, d: 4]
    
    iex> Keyword.merge([a: 1, b: 2], [a: 3, d: 4, a: 5], fn :a, v1, v2 ->
    ...>   v1 + v2
    ...> end)
    [b: 2, a: 4, d: 4, a: 5]
    
    iex> Keyword.merge([a: 1, b: 2, a: 3], [a: 3, d: 4, a: 5], fn :a, v1, v2 ->
    ...>   v1 + v2
    ...> end)
    [b: 2, a: 4, d: 4, a: 8]
    
    iex> Keyword.merge([a: 1, b: 2], [:a, :b], fn :a, v1, v2 ->
    ...>   v1 + v2
    ...> end)
    ** (ArgumentError) expected a keyword list as the second argument, got: [:a, :b]


### new()

```elixir
@spec new() :: []
```

Returns an empty keyword list, i.e. an empty list.

#### Examples

    iex> Keyword.new()
    []


### new(pairs)

```elixir
@spec new(Enumerable.t()) :: t()
```

Creates a keyword list from an enumerable.

Removes duplicate entries and the last one prevails.
Unlike `Enum.into(enumerable, [])`, `Keyword.new(enumerable)`
guarantees the keys are unique.

#### Examples

    iex> Keyword.new([{:b, 1}, {:a, 2}])
    [b: 1, a: 2]
    
    iex> Keyword.new([{:a, 1}, {:a, 2}, {:a, 3}])
    [a: 3]


### new(pairs, transform)

```elixir
@spec new(Enumerable.t(), (term() -&gt; {key(), value()})) :: t()
```

Creates a keyword list from an enumerable via the transformation function.

Removes duplicate entries and the last one prevails.
Unlike `Enum.into(enumerable, [], fun)`,
`Keyword.new(enumerable, fun)` guarantees the keys are unique.

#### Examples

    iex> Keyword.new([:a, :b], fn x -> {x, x} end)
    [a: :a, b: :b]


### pop(keywords, key, default \\ nil)

```elixir
@spec pop(t(), key(), default()) :: {value() | default(), t()}
```

Returns the first value for `key` and removes all associated entries in the keyword list.

It returns a tuple where the first element is the first value for `key` and the
second element is a keyword list with all entries associated with `key` removed.
If the `key` is not present in the keyword list, it returns `{default, keyword_list}`.

If you don't want to remove all the entries associated with `key` use `pop_first/3`
instead, which will remove only the first entry.

#### Examples

    iex> Keyword.pop([a: 1], :a)
    {1, []}
    iex> Keyword.pop([a: 1], :b)
    {nil, [a: 1]}
    iex> Keyword.pop([a: 1], :b, 3)
    {3, [a: 1]}
    iex> Keyword.pop([a: 1, a: 2], :a)
    {1, []}


### pop!(keywords, key)
*(since 1.10.0)* 
```elixir
@spec pop!(t(), key()) :: {value(), t()}
```

Returns the first value for `key` and removes all associated entries in the keyword list,
raising if `key` is not present.

This function behaves like `pop/3`, but raises in case the `key` is not present in the
given `keywords`.

#### Examples

    iex> Keyword.pop!([a: 1], :a)
    {1, []}
    iex> Keyword.pop!([a: 1, a: 2], :a)
    {1, []}
    iex> Keyword.pop!([a: 1], :b)
    ** (KeyError) key :b not found in: [a: 1]


### pop_first(keywords, key, default \\ nil)

```elixir
@spec pop_first(t(), key(), default()) :: {value() | default(), t()}
```

Returns and removes the first value associated with `key` in the keyword list.

Keeps duplicate keys in the resulting keyword list.

#### Examples

    iex> Keyword.pop_first([a: 1], :a)
    {1, []}
    iex> Keyword.pop_first([a: 1], :b)
    {nil, [a: 1]}
    iex> Keyword.pop_first([a: 1], :b, 3)
    {3, [a: 1]}
    iex> Keyword.pop_first([a: 1, a: 2], :a)
    {1, [a: 2]}


### pop_lazy(keywords, key, fun)

```elixir
@spec pop_lazy(t(), key(), (-&gt; value())) :: {value(), t()}
```

Lazily returns and removes all values associated with `key` in the keyword list.

This is useful if the default value is very expensive to calculate or
generally difficult to set up and tear down again.

Removes all duplicate keys. See `pop_first/3` for removing only the first entry.

#### Examples

    iex> keyword = [a: 1]
    iex> fun = fn ->
    ...>   # some expensive operation here
    ...>   13
    ...> end
    iex> Keyword.pop_lazy(keyword, :a, fun)
    {1, []}
    iex> Keyword.pop_lazy(keyword, :b, fun)
    {13, [a: 1]}


### pop_values(keywords, key)
*(since 1.10.0)* 
```elixir
@spec pop_values(t(), key()) :: {[value()], t()}
```

Returns all values for `key` and removes all associated entries in the keyword list.

It returns a tuple where the first element is a list of values for `key` and the
second element is a keyword list with all entries associated with `key` removed.
If the `key` is not present in the keyword list, it returns `{[], keyword_list}`.

If you don't want to remove all the entries associated with `key` use `pop_first/3`
instead, which will remove only the first entry.

#### Examples

    iex> Keyword.pop_values([a: 1], :a)
    {[1], []}
    iex> Keyword.pop_values([a: 1], :b)
    {[], [a: 1]}
    iex> Keyword.pop_values([a: 1, a: 2], :a)
    {[1, 2], []}


### put(keywords, key, value)

```elixir
@spec put(t(), key(), value()) :: t()
```

Puts the given `value` under the specified `key`.

If a value under `key` already exists, it overrides the value
and removes all duplicate entries.

#### Examples

    iex> Keyword.put([a: 1], :b, 2)
    [b: 2, a: 1]
    iex> Keyword.put([a: 1, b: 2], :a, 3)
    [a: 3, b: 2]
    iex> Keyword.put([a: 1, b: 2, a: 4], :a, 3)
    [a: 3, b: 2]


### put_new(keywords, key, value)

```elixir
@spec put_new(t(), key(), value()) :: t()
```

Puts the given `value` under `key`, unless the entry `key` already exists.

#### Examples

    iex> Keyword.put_new([a: 1], :b, 2)
    [b: 2, a: 1]
    iex> Keyword.put_new([a: 1, b: 2], :a, 3)
    [a: 1, b: 2]


### put_new_lazy(keywords, key, fun)

```elixir
@spec put_new_lazy(t(), key(), (-&gt; value())) :: t()
```

Evaluates `fun` and puts the result under `key`
in keyword list unless `key` is already present.

This is useful if the value is very expensive to calculate or
generally difficult to set up and tear down again.

#### Examples

    iex> keyword = [a: 1]
    iex> fun = fn ->
    ...>   # some expensive operation here
    ...>   13
    ...> end
    iex> Keyword.put_new_lazy(keyword, :a, fun)
    [a: 1]
    iex> Keyword.put_new_lazy(keyword, :b, fun)
    [b: 13, a: 1]


### reject(keywords, fun)
*(since 1.13.0)* 
```elixir
@spec reject(t(), ({key(), value()} -&gt; as_boolean(term()))) :: t()
```

Returns a keyword list excluding the entries from `keywords`
for which the function `fun` returns a truthy value.

See also `filter/2`.

#### Examples

    iex> Keyword.reject([one: 1, two: 2, three: 3], fn {_key, val} -> rem(val, 2) == 1 end)
    [two: 2]


### replace(keywords, key, value)
*(since 1.11.0)* 
```elixir
@spec replace(t(), key(), value()) :: t()
```

Puts a value under `key` only if the `key` already exists in `keywords`.

In case a key exists multiple times in the keyword list,
it removes later occurrences.

#### Examples

    iex> Keyword.replace([a: 1, b: 2, a: 4], :a, 3)
    [a: 3, b: 2]
    
    iex> Keyword.replace([a: 1], :b, 2)
    [a: 1]


### replace!(keywords, key, value)
*(since 1.5.0)* 
```elixir
@spec replace!(t(), key(), value()) :: t()
```

Puts a value under `key` only if the `key` already exists in `keywords`.

If `key` is not present in `keywords`, it raises a `KeyError`.

#### Examples

    iex> Keyword.replace!([a: 1, b: 2, a: 3], :a, :new)
    [a: :new, b: 2]
    iex> Keyword.replace!([a: 1, b: 2, c: 3, b: 4], :b, :new)
    [a: 1, b: :new, c: 3]
    
    iex> Keyword.replace!([a: 1], :b, 2)
    ** (KeyError) key :b not found in: [a: 1]


### replace_lazy(keywords, key, fun)
*(since 1.14.0)* 
```elixir
@spec replace_lazy(t(), key(), (existing_value :: value() -&gt; new_value :: value())) ::
  t()
```

Replaces the value under `key` using the given function only if
`key` already exists in `keywords`.

In comparison to `replace/3`, this can be useful when it's expensive to calculate the value.

If `key` does not exist, the original keyword list is returned unchanged.

#### Examples

    iex> Keyword.replace_lazy([a: 1, b: 2], :a, fn v -> v * 4 end)
    [a: 4, b: 2]
    
    iex> Keyword.replace_lazy([a: 2, b: 2, a: 1], :a, fn v -> v * 4 end)
    [a: 8, b: 2]
    
    iex> Keyword.replace_lazy([a: 1, b: 2], :c, fn v -> v * 4 end)
    [a: 1, b: 2]


### split(keywords, keys)

```elixir
@spec split(t(), [key()]) :: {t(), t()}
```

Takes all entries corresponding to the given `keys` and extracts them into a
separate keyword list.

Returns a tuple with the new list and the old list with removed keys.

Ignores keys for which there are no entries in the keyword list.

Entries with duplicate keys end up in the same keyword list.

#### Examples

    iex> Keyword.split([a: 1, b: 2, c: 3], [:a, :c, :e])
    {[a: 1, c: 3], [b: 2]}
    iex> Keyword.split([a: 1, b: 2, c: 3, a: 4], [:a, :c, :e])
    {[a: 1, c: 3, a: 4], [b: 2]}


### split_with(keywords, fun)
*(since 1.15.0)* 
```elixir
@spec split_with(t(), ({key(), value()} -&gt; as_boolean(term()))) :: {t(), t()}
```

Splits the `keywords` into two keyword lists according to the given function
`fun`.

The provided `fun` receives each `{key, value}` pair in the `keywords` as its only
argument. Returns a tuple with the first keyword list containing all the
elements in `keywords` for which applying `fun` returned a truthy value, and
a second keyword list with all the elements for which applying `fun` returned
a falsy value (`false` or `nil`).

#### Examples

    iex> Keyword.split_with([a: 1, b: 2, c: 3], fn {_k, v} -> rem(v, 2) == 0 end)
    {[b: 2], [a: 1, c: 3]}
    
    iex> Keyword.split_with([a: 1, b: 2, c: 3, b: 4], fn {_k, v} -> rem(v, 2) == 0 end)
    {[b: 2, b: 4], [a: 1, c: 3]}
    
    iex> Keyword.split_with([a: 1, b: 2, c: 3, b: 4], fn {k, v} -> k in [:a, :c] and rem(v, 2) == 0 end)
    {[], [a: 1, b: 2, c: 3, b: 4]}
    
    iex> Keyword.split_with([], fn {_k, v} -> rem(v, 2) == 0 end)
    {[], []}


### take(keywords, keys)

```elixir
@spec take(t(), [key()]) :: t()
```

Takes all entries corresponding to the given `keys` and returns them as a new
keyword list.

Preserves duplicate keys in the new keyword list.

#### Examples

    iex> Keyword.take([a: 1, b: 2, c: 3], [:a, :c, :e])
    [a: 1, c: 3]
    iex> Keyword.take([a: 1, b: 2, c: 3, a: 5], [:a, :c, :e])
    [a: 1, c: 3, a: 5]


### to_list(keywords)

```elixir
@spec to_list(t()) :: t()
```

Returns the keyword list itself.

#### Examples

    iex> Keyword.to_list(a: 1)
    [a: 1]


### update(keywords, key, default, fun)

```elixir
@spec update(t(), key(), default :: value(), (existing_value :: value() -&gt;
                                          new_value :: value())) :: t()
```

Updates the value under `key` in `keywords` using the given function.

If the `key` does not exist, it inserts the given `default` value.
Does not pass the `default` value through the update function.

Removes all duplicate keys and only updates the first one.

#### Examples

    iex> Keyword.update([a: 1], :a, 13, fn existing_value -> existing_value * 2 end)
    [a: 2]
    
    iex> Keyword.update([a: 1, a: 2], :a, 13, fn existing_value -> existing_value * 2 end)
    [a: 2]
    
    iex> Keyword.update([a: 1], :b, 11, fn existing_value -> existing_value * 2 end)
    [a: 1, b: 11]


### update!(keywords, key, fun)

```elixir
@spec update!(t(), key(), (current_value :: value() -&gt; new_value :: value())) :: t()
```

Updates the value under `key` using the given function.

Raises `KeyError` if the `key` does not exist.

Removes all duplicate keys and only updates the first one.

#### Examples

    iex> Keyword.update!([a: 1, b: 2, a: 3], :a, &(&1 * 2))
    [a: 2, b: 2]
    iex> Keyword.update!([a: 1, b: 2, c: 3], :b, &(&1 * 2))
    [a: 1, b: 4, c: 3]
    
    iex> Keyword.update!([a: 1], :b, &(&1 * 2))
    ** (KeyError) key :b not found in: [a: 1]


### validate(keyword, values)
*(since 1.13.0)* 
```elixir
@spec validate(
  keyword(),
  values :: [atom() | {atom(), term()}]
) :: {:ok, keyword()} | {:error, [atom()]}
```

Ensures the given `keyword` has only the keys given in `values`.

The second argument must be a list of atoms, specifying
a given key, or tuples specifying a key and a default value.

If the keyword list has only the given keys, it returns
`{:ok, keyword}` with default values applied. Otherwise it
returns `{:error, invalid_keys}` with invalid keys.

See also: `validate!/2`.

#### Examples

    iex> {:ok, result} = Keyword.validate([], [one: 1, two: 2])
    iex> Enum.sort(result)
    [one: 1, two: 2]
    
    iex> {:ok, result} = Keyword.validate([two: 3], [one: 1, two: 2])
    iex> Enum.sort(result)
    [one: 1, two: 3]

If atoms are given, they are supported as keys but do not
provide a default value:

    iex> {:ok, result} = Keyword.validate([], [:one, two: 2])
    iex> Enum.sort(result)
    [two: 2]
    
    iex> {:ok, result} = Keyword.validate([one: 1], [:one, two: 2])
    iex> Enum.sort(result)
    [one: 1, two: 2]

Passing unknown keys returns an error:

    iex> Keyword.validate([three: 3, four: 4], [one: 1, two: 2])
    {:error, [:four, :three]}

Passing the same key multiple times also errors:

    iex> Keyword.validate([one: 1, two: 2, one: 1], [:one, :two])
    {:error, [:one]}


### validate!(keyword, values)
*(since 1.13.0)* 
```elixir
@spec validate!(
  keyword(),
  values :: [atom() | {atom(), term()}]
) :: keyword()
```

Similar to `validate/2` but returns the keyword or raises an error.

#### Examples

    iex> Keyword.validate!([], [one: 1, two: 2]) |> Enum.sort()
    [one: 1, two: 2]
    iex> Keyword.validate!([two: 3], [one: 1, two: 2]) |> Enum.sort()
    [one: 1, two: 3]

If atoms are given, they are supported as keys but do not
provide a default value:

    iex> Keyword.validate!([], [:one, two: 2]) |> Enum.sort()
    [two: 2]
    iex> Keyword.validate!([one: 1], [:one, two: 2]) |> Enum.sort()
    [one: 1, two: 2]

Passing unknown keys raises an error:

    iex> Keyword.validate!([three: 3], [one: 1, two: 2])
    ** (ArgumentError) unknown keys [:three] in [three: 3], the allowed keys are: [:one, :two]

Passing the same key multiple times also errors:

    iex> Keyword.validate!([one: 1, two: 2, one: 1], [:one, :two])
    ** (ArgumentError) duplicate keys [:one] in [one: 1, two: 2, one: 1]


### values(keywords)

```elixir
@spec values(t()) :: [value()]
```

Returns all values from the keyword list.

Keeps values from duplicate keys in the resulting list of values.

#### Examples

    iex> Keyword.values(a: 1, b: 2)
    [1, 2]
    iex> Keyword.values(a: 1, b: 2, a: 3)
    [1, 2, 3]




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
