# Access behaviour
(Elixir v1.18.0-dev)

Key-based access to data structures.

The `Access` module defines a behaviour for dynamically accessing
keys of any type in a data structure via the `data[key]` syntax.

`Access` supports keyword lists (`Keyword`) and maps (`Map`) out
of the box. Keywords supports only atoms keys, keys for maps can
be of any type. Both return `nil` if the key does not exist:

    iex> keywords = [a: 1, b: 2]
    iex> keywords[:a]
    1
    iex> keywords[:c]
    nil
    
    iex> map = %{a: 1, b: 2}
    iex> map[:a]
    1
    
    iex> star_ratings = %{1.0 => "★", 1.5 => "★☆", 2.0 => "★★"}
    iex> star_ratings[1.5]
    "★☆"

This syntax is very convenient as it can be nested arbitrarily:

    iex> keywords = [a: 1, b: 2]
    iex> keywords[:c][:unknown]
    nil

This works because accessing anything on a `nil` value, returns
`nil` itself:

    iex> nil[:a]
    nil

## Maps and structs

While the access syntax is allowed in maps via `map[key]`,
if your map is made of predefined atom keys, you should prefer
to access those atom keys with `map.key` instead of `map[key]`,
as `map.key` will raise if the key is missing (which is not
supposed to happen if the keys are predefined) or if `map` is
`nil`.

Similarly, since structs are maps and structs have predefined
keys, they only allow the `struct.key` syntax and they do not
allow the `struct[key]` access syntax.

In other words, the `map[key]` syntax is loose, returning `nil`
for missing keys, while the `map.key` syntax is strict, raising
for both nil values and missing keys.

To bridge this gap, Elixir provides the `get_in/1` and `get_in/2`
functions, which are capable of traversing nested data structures,
even in the presence of `nil`s:

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> get_in(users["john"].age)
    27
    iex> get_in(users["unknown"].age)
    nil

Notice how, even if no user was found, `get_in/1` returned `nil`.
Outside of `get_in/1`, trying to access the field `.age` on `nil`
would raise.

The `get_in/2` function takes one step further by allowing
different accessors to be mixed in. For example, given a user
map with the `:name` and `:languages` keys, here is how to
access the name of all programming languages:

```
  iex> languages = [
  ...>   %{name: "elixir", type: :functional},
  ...>   %{name: "c", type: :procedural}
  ...> ]
  iex> user = %{name: "john", languages: languages}
  iex> get_in(user, [:languages, Access.all(), :name])
  ["elixir", "c"]
```

This module provides convenience functions for traversing other
structures, like tuples and lists. As we will see next, they can
even be used to update nested data structures.

If you want to learn more about the dual nature of maps in Elixir,
as they can be either for structured data or as a key-value store,
see the `Map` module.

## Updating nested data structures

The access syntax can also be used with the `Kernel.put_in/2`,
`Kernel.update_in/2`, `Kernel.get_and_update_in/2`, and `Kernel.pop_in/1`
macros to further manipulate values in nested data structures:

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> put_in(users["john"].age, 28)
    %{"john" => %{age: 28}, "meg" => %{age: 23}}

As shown in the previous section, you can also use the
`Kernel.put_in/3`, `Kernel.update_in/3`, `Kernel.pop_in/2`, and
`Kernel.get_and_update_in/3` functions to provide nested
custom accessors. For instance, given a user map with the
`:name` and `:languages` keys, here is how to deeply traverse
the map and convert all language names to uppercase:

    iex> languages = [
    ...>   %{name: "elixir", type: :functional},
    ...>   %{name: "c", type: :procedural}
    ...> ]
    iex> user = %{name: "john", languages: languages}
    iex> update_in(user, [:languages, Access.all(), :name], &String.upcase/1)
    %{
      name: "john",
      languages: [
        %{name: "ELIXIR", type: :functional},
        %{name: "C", type: :procedural}
      ]
    }

See the functions `key/1`, `key!/1`, `elem/1`, and `all/0` for
some of the available accessors.

## Types

### access_fun(data, current_value)

```elixir
@type access_fun(data, current_value) ::
  get_fun(data) | get_and_update_fun(data, current_value)
```



### container()

```elixir
@type container() :: keyword() | struct() | map()
```



### get_and_update_fun(data, current_value)

```elixir
@type get_and_update_fun(data, current_value) :: (:get_and_update,
                                            data,
                                            (term() -&gt; term()) -&gt;
                                              {current_value,
                                               new_data :: container()}
                                              | :pop)
```



### get_fun(data)

```elixir
@type get_fun(data) :: (:get, data, (term() -&gt; term()) -&gt; new_data :: container())
```



### key()

```elixir
@type key() :: any()
```



### nil_container()

```elixir
@type nil_container() :: nil
```



### t()

```elixir
@type t() :: container() | nil_container() | any()
```



### value()

```elixir
@type value() :: any()
```



## Callbacks

### fetch(term, key)

```elixir
@callback fetch(term :: t(), key()) :: {:ok, value()} | :error
```

Invoked in order to access the value stored under `key` in the given term `term`.

This function should return `{:ok, value}` where `value` is the value under
`key` if the key exists in the term, or `:error` if the key does not exist in
the term.

Many of the functions defined in the `Access` module internally call this
function. This function is also used when the square-brackets access syntax
(`structure[key]`) is used: the `fetch/2` callback implemented by the module
that defines the `structure` struct is invoked and if it returns `{:ok, value}` then `value` is returned, or if it returns `:error` then `nil` is
returned.

See the `Map.fetch/2` and `Keyword.fetch/2` implementations for examples of
how to implement this callback.

### get_and_update(data, key, function)

```elixir
@callback get_and_update(data, key(), (value() | nil -&gt;
                               {current_value, new_value :: value()} | :pop)) ::
  {current_value, new_data :: data}
when current_value: value(), data: container()
```

Invoked in order to access the value under `key` and update it at the same time.

The implementation of this callback should invoke `fun` with the value under
`key` in the passed structure `data`, or with `nil` if `key` is not present in it.
This function must return either `{current_value, new_value}` or `:pop`.

If the passed function returns `{current_value, new_value}`,
the return value of this callback should be `{current_value, new_data}`, where:

- `current_value` is the retrieved value (which can be operated on before being returned)

- `new_value` is the new value to be stored under `key`

- `new_data` is `data` after updating the value of `key` with `new_value`.

If the passed function returns `:pop`, the return value of this callback
must be `{value, new_data}` where `value` is the value under `key`
(or `nil` if not present) and `new_data` is `data` without `key`.

See the implementations of `Map.get_and_update/3` or `Keyword.get_and_update/3`
for more examples.

### pop(data, key)

```elixir
@callback pop(data, key()) :: {value(), data} when data: container()
```

Invoked to "pop" the value under `key` out of the given data structure.

When `key` exists in the given structure `data`, the implementation should
return a `{value, new_data}` tuple where `value` is the value that was under
`key` and `new_data` is `term` without `key`.

When `key` is not present in the given structure, a tuple `{value, data}`
should be returned, where `value` is implementation-defined.

See the implementations for `Map.pop/3` or `Keyword.pop/3` for more examples.

## Functions

### all()

```elixir
@spec all() :: access_fun(data :: list(), current_value :: list())
```

Returns a function that accesses all the elements in a list.

The returned function is typically passed as an accessor to `Kernel.get_in/2`,
`Kernel.get_and_update_in/3`, and friends.

#### Examples

    iex> list = [%{name: "john"}, %{name: "mary"}]
    iex> get_in(list, [Access.all(), :name])
    ["john", "mary"]
    iex> get_and_update_in(list, [Access.all(), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {["john", "mary"], [%{name: "JOHN"}, %{name: "MARY"}]}
    iex> pop_in(list, [Access.all(), :name])
    {["john", "mary"], [%{}, %{}]}

Here is an example that traverses the list dropping even
numbers and multiplying odd numbers by 2:

    iex> require Integer
    iex> get_and_update_in([1, 2, 3, 4, 5], [Access.all()], fn num ->
    ...>   if Integer.is_even(num), do: :pop, else: {num, num * 2}
    ...> end)
    {[1, 2, 3, 4, 5], [2, 6, 10]}

An error is raised if the accessed structure is not a list:

    iex> get_in(%{}, [Access.all()])
    ** (RuntimeError) Access.all/0 expected a list, got: %{}

### at(index)

```elixir
@spec at(integer()) :: access_fun(data :: list(), current_value :: term())
```

Returns a function that accesses the element at `index` (zero based) of a list.

Keep in mind that index lookups in lists take linear time: the larger the list,
the longer it will take to access its index. Therefore index-based operations
are generally avoided in favor of other functions in the `Enum` module.

The returned function is typically passed as an accessor to `Kernel.get_in/2`,
`Kernel.get_and_update_in/3`, and friends.

#### Examples

    iex> list = [%{name: "john"}, %{name: "mary"}]
    iex> get_in(list, [Access.at(1), :name])
    "mary"
    iex> get_in(list, [Access.at(-1), :name])
    "mary"
    iex> get_and_update_in(list, [Access.at(0), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {"john", [%{name: "JOHN"}, %{name: "mary"}]}
    iex> get_and_update_in(list, [Access.at(-1), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {"mary", [%{name: "john"}, %{name: "MARY"}]}

`at/1` can also be used to pop elements out of a list or
a key inside of a list:

    iex> list = [%{name: "john"}, %{name: "mary"}]
    iex> pop_in(list, [Access.at(0)])
    {%{name: "john"}, [%{name: "mary"}]}
    iex> pop_in(list, [Access.at(0), :name])
    {"john", [%{}, %{name: "mary"}]}

When the index is out of bounds, `nil` is returned and the update function is never called:

    iex> list = [%{name: "john"}, %{name: "mary"}]
    iex> get_in(list, [Access.at(10), :name])
    nil
    iex> get_and_update_in(list, [Access.at(10), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {nil, [%{name: "john"}, %{name: "mary"}]}

An error is raised if the accessed structure is not a list:

    iex> get_in(%{}, [Access.at(1)])
    ** (RuntimeError) Access.at/1 expected a list, got: %{}

### at!(index)
*(since 1.11.0)* 
```elixir
@spec at!(integer()) :: access_fun(data :: list(), current_value :: term())
```

Same as `at/1` except that it raises `Enum.OutOfBoundsError`
if the given index is out of bounds.

#### Examples

    iex> get_in([:a, :b, :c], [Access.at!(2)])
    :c
    iex> get_in([:a, :b, :c], [Access.at!(3)])
    ** (Enum.OutOfBoundsError) out of bounds error

### elem(index)

```elixir
@spec elem(non_neg_integer()) :: access_fun(data :: tuple(), current_value :: term())
```

Returns a function that accesses the element at the given index in a tuple.

The returned function is typically passed as an accessor to `Kernel.get_in/2`,
`Kernel.get_and_update_in/3`, and friends.

The returned function raises if `index` is out of bounds.

Note that popping elements out of tuples is not possible and raises an
error.

#### Examples

    iex> map = %{user: {"john", 27}}
    iex> get_in(map, [:user, Access.elem(0)])
    "john"
    iex> get_and_update_in(map, [:user, Access.elem(0)], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {"john", %{user: {"JOHN", 27}}}
    iex> pop_in(map, [:user, Access.elem(0)])
    ** (RuntimeError) cannot pop data from a tuple

An error is raised if the accessed structure is not a tuple:

    iex> get_in(%{}, [Access.elem(0)])
    ** (RuntimeError) Access.elem/1 expected a tuple, got: %{}

### fetch(container, key)

```elixir
@spec fetch(container(), term()) :: {:ok, term()} | :error
@spec fetch(nil_container(), any()) :: :error
```

Fetches the value for the given key in a container (a map, keyword
list, or struct that implements the `Access` behaviour).

Returns `{:ok, value}` where `value` is the value under `key` if there is such
a key, or `:error` if `key` is not found.

#### Examples

    iex> Access.fetch(%{name: "meg", age: 26}, :name)
    {:ok, "meg"}
    
    iex> Access.fetch([ordered: true, on_timeout: :exit], :timeout)
    :error

### fetch!(container, key)
*(since 1.10.0)* 
```elixir
@spec fetch!(container(), term()) :: term()
```

Same as `fetch/2` but returns the value directly,
or raises a `KeyError` exception if `key` is not found.

#### Examples

    iex> Access.fetch!(%{name: "meg", age: 26}, :name)
    "meg"

### filter(func)
*(since 1.6.0)* 
```elixir
@spec filter((term() -&gt; boolean())) ::
  access_fun(data :: list(), current_value :: list())
```

Returns a function that accesses all elements of a list that match the provided predicate.

The returned function is typically passed as an accessor to `Kernel.get_in/2`,
`Kernel.get_and_update_in/3`, and friends.

#### Examples

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}]
    iex> get_in(list, [Access.filter(&(&1.salary > 20)), :name])
    ["francine"]
    iex> get_and_update_in(list, [Access.filter(&(&1.salary <= 20)), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {["john"], [%{name: "JOHN", salary: 10}, %{name: "francine", salary: 30}]}

`filter/1` can also be used to pop elements out of a list or
a key inside of a list:

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}]
    iex> pop_in(list, [Access.filter(&(&1.salary >= 20))])
    {[%{name: "francine", salary: 30}], [%{name: "john", salary: 10}]}
    iex> pop_in(list, [Access.filter(&(&1.salary >= 20)), :name])
    {["francine"], [%{name: "john", salary: 10}, %{salary: 30}]}

When no match is found, an empty list is returned and the update function is never called

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}]
    iex> get_in(list, [Access.filter(&(&1.salary >= 50)), :name])
    []
    iex> get_and_update_in(list, [Access.filter(&(&1.salary >= 50)), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {[], [%{name: "john", salary: 10}, %{name: "francine", salary: 30}]}

An error is raised if the predicate is not a function or is of the incorrect arity:

    iex> get_in([], [Access.filter(5)])
    ** (FunctionClauseError) no function clause matching in Access.filter/1

An error is raised if the accessed structure is not a list:

    iex> get_in(%{}, [Access.filter(fn a -> a == 10 end)])
    ** (RuntimeError) Access.filter/1 expected a list, got: %{}

### find(predicate)
*(since 1.17.0)* 
```elixir
@spec find((term() -&gt; as_boolean(term()))) ::
  access_fun(data :: list(), current_value :: term())
```

Returns a function that accesses the first element of a list that matches the provided predicate.

The returned function is typically passed as an accessor to `Kernel.get_in/2`,
`Kernel.get_and_update_in/3`, and friends.

#### Examples

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}]
    iex> get_in(list, [Access.find(&(&1.salary > 20)), :name])
    "francine"
    iex>  get_and_update_in(list, [Access.find(&(&1.salary <= 40)), :name], fn prev ->
    ...> {prev, String.upcase(prev)}
    ...>  end)
    {"john", [%{name: "JOHN", salary: 10}, %{name: "francine", salary: 30}]}

`find/1` can also be used to pop the first found element out of a list or
a key inside of a list:

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}]
    iex> pop_in(list, [Access.find(&(&1.salary <= 40))])
    {%{name: "john", salary: 10}, [%{name: "francine", salary: 30}]}

When no match is found, nil is returned and the update function is never called

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}]
    iex> get_in(list, [Access.find(&(&1.salary >= 50)), :name])
    nil
    iex> get_and_update_in(list, [Access.find(&(&1.salary >= 50)), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {nil, [%{name: "john", salary: 10}, %{name: "francine", salary: 30}]}

An error is raised if the predicate is not a function or is of the incorrect arity:

    iex> get_in([], [Access.find(5)])
    ** (FunctionClauseError) no function clause matching in Access.find/1

An error is raised if the accessed structure is not a list:

    iex>  get_in(%{}, [Access.find(fn a -> a == 10 end)])
    ** (RuntimeError) Access.find/1 expected a list, got: %{}

### get(container, key, default \\ nil)

```elixir
@spec get(container(), term(), term()) :: term()
@spec get(nil_container(), any(), default) :: default when default: var
```

Gets the value for the given key in a container (a map, keyword
list, or struct that implements the `Access` behaviour).

Returns the value under `key` if there is such a key, or `default` if `key` is
not found.

#### Examples

    iex> Access.get(%{name: "john"}, :name, "default name")
    "john"
    iex> Access.get(%{name: "john"}, :age, 25)
    25
    
    iex> Access.get([ordered: true], :timeout)
    nil

### get_and_update(container, key, fun)

```elixir
@spec get_and_update(data, key(), (value() | nil -&gt;
                               {current_value, new_value :: value()} | :pop)) ::
  {current_value, new_data :: data}
when data: container(), current_value: var
```

Gets and updates the given key in a `container` (a map, a keyword list,
a struct that implements the `Access` behaviour).

The `fun` argument receives the value of `key` (or `nil` if `key` is not
present in `container`) and must return a two-element tuple `{current_value, new_value}`:
the "get" value `current_value` (the retrieved value, which can be operated on before
being returned) and the new value to be stored under `key` (`new_value`).
`fun` may also return `:pop`, which means the current value
should be removed from the container and returned.

The returned value is a two-element tuple with the "get" value returned by
`fun` and a new container with the updated value under `key`.

#### Examples

    iex> Access.get_and_update([a: 1], :a, fn current_value ->
    ...>   {current_value, current_value + 1}
    ...> end)
    {1, [a: 2]}

### key(key, default \\ nil)

```elixir
@spec key(key(), term()) ::
  access_fun(data :: struct() | map(), current_value :: term())
```

Returns a function that accesses the given key in a map/struct.

The returned function is typically passed as an accessor to `Kernel.get_in/2`,
`Kernel.get_and_update_in/3`, and friends.

The returned function uses the default value if the key does not exist.
This can be used to specify defaults and safely traverse missing keys:

    iex> get_in(%{}, [Access.key(:user, %{}), Access.key(:name, "meg")])
    "meg"

Such is also useful when using update functions, allowing us to introduce
values as we traverse the data structure for updates:

    iex> put_in(%{}, [Access.key(:user, %{}), Access.key(:name)], "Mary")
    %{user: %{name: "Mary"}}

#### Examples

    iex> map = %{user: %{name: "john"}}
    iex> get_in(map, [Access.key(:unknown, %{}), Access.key(:name, "john")])
    "john"
    iex> get_and_update_in(map, [Access.key(:user), Access.key(:name)], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {"john", %{user: %{name: "JOHN"}}}
    iex> pop_in(map, [Access.key(:user), Access.key(:name)])
    {"john", %{user: %{}}}

An error is raised if the accessed structure is not a map or a struct:

    iex> get_in([], [Access.key(:foo)])
    ** (BadMapError) expected a map, got: []

### key!(key)

```elixir
@spec key!(key()) :: access_fun(data :: struct() | map(), current_value :: term())
```

Returns a function that accesses the given key in a map/struct.

The returned function is typically passed as an accessor to `Kernel.get_in/2`,
`Kernel.get_and_update_in/3`, and friends.

Similar to `key/2`, but the returned function raises if the key does not exist.

#### Examples

    iex> map = %{user: %{name: "john"}}
    iex> get_in(map, [Access.key!(:user), Access.key!(:name)])
    "john"
    iex> get_and_update_in(map, [Access.key!(:user), Access.key!(:name)], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {"john", %{user: %{name: "JOHN"}}}
    iex> pop_in(map, [Access.key!(:user), Access.key!(:name)])
    {"john", %{user: %{}}}
    iex> get_in(map, [Access.key!(:user), Access.key!(:unknown)])
    ** (KeyError) key :unknown not found in: %{name: "john"}

The examples above could be partially written as:

    iex> map = %{user: %{name: "john"}}
    iex> map.user.name
    "john"
    iex> get_and_update_in(map.user.name, fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {"john", %{user: %{name: "JOHN"}}}

However, it is not possible to remove fields using the dot notation,
as it is implied those fields must also be present. In any case,
`Access.key!/1` is useful when the key is not known in advance
and must be accessed dynamically.

An error is raised if the accessed structure is not a map/struct:

    iex> get_in([], [Access.key!(:foo)])
    ** (RuntimeError) Access.key!/1 expected a map/struct, got: []

### pop(container, key)

```elixir
@spec pop(data, key()) :: {value(), data} when data: container()
```

Removes the entry with a given key from a container (a map, keyword
list, or struct that implements the `Access` behaviour).

Returns a tuple containing the value associated with the key and the
updated container. `nil` is returned for the value if the key isn't
in the container.

#### Examples

With a map:

    iex> Access.pop(%{name: "Elixir", creator: "Valim"}, :name)
    {"Elixir", %{creator: "Valim"}}

A keyword list:

    iex> Access.pop([name: "Elixir", creator: "Valim"], :name)
    {"Elixir", [creator: "Valim"]}

An unknown key:

    iex> Access.pop(%{name: "Elixir", creator: "Valim"}, :year)
    {nil, %{creator: "Valim", name: "Elixir"}}

### slice(range)
*(since 1.14)* 
```elixir
@spec slice(Range.t()) :: access_fun(data :: list(), current_value :: list())
```

Returns a function that accesses all items of a list that are within the provided range.

The range will be normalized following the same rules from `Enum.slice/2`.

The returned function is typically passed as an accessor to `Kernel.get_in/2`,
`Kernel.get_and_update_in/3`, and friends.

#### Examples

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}, %{name: "vitor", salary: 25}]
    iex> get_in(list, [Access.slice(1..2), :name])
    ["francine", "vitor"]
    iex> get_and_update_in(list, [Access.slice(1..3//2), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {["francine"], [%{name: "john", salary: 10}, %{name: "FRANCINE", salary: 30}, %{name: "vitor", salary: 25}]}

`slice/1` can also be used to pop elements out of a list or
a key inside of a list:

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}, %{name: "vitor", salary: 25}]
    iex> pop_in(list, [Access.slice(-2..-1)])
    {[%{name: "francine", salary: 30}, %{name: "vitor", salary: 25}], [%{name: "john", salary: 10}]}
    iex> pop_in(list, [Access.slice(-2..-1), :name])
    {["francine", "vitor"], [%{name: "john", salary: 10}, %{salary: 30}, %{salary: 25}]}

When no match is found, an empty list is returned and the update function is never called

    iex> list = [%{name: "john", salary: 10}, %{name: "francine", salary: 30}, %{name: "vitor", salary: 25}]
    iex> get_in(list, [Access.slice(5..10//2), :name])
    []
    iex> get_and_update_in(list, [Access.slice(5..10//2), :name], fn prev ->
    ...>   {prev, String.upcase(prev)}
    ...> end)
    {[], [%{name: "john", salary: 10}, %{name: "francine", salary: 30}, %{name: "vitor", salary: 25}]}

An error is raised if the accessed structure is not a list:

    iex> get_in(%{}, [Access.slice(2..10//3)])
    ** (ArgumentError) Access.slice/1 expected a list, got: %{}

An error is raised if the step of the range is negative:

    iex> get_in([], [Access.slice(2..10//-1)])
    ** (ArgumentError) Access.slice/1 does not accept ranges with negative steps, got: 2..10//-1



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
