# MapSet 
(Elixir v1.18.0-dev)

Functions that work on sets.

A set is a data structure that can contain unique elements of any kind,
without any particular order. `MapSet` is the "go to" set data structure in Elixir.

A set can be constructed using `MapSet.new/0`:

    iex> MapSet.new()
    MapSet.new([])

Elements in a set don't have to be of the same type and they can be
populated from an [enumerable](\`t:Enumerable.t/0\`) using `MapSet.new/1`:

    iex> MapSet.new([1, :two, {"three"}])
    MapSet.new([1, :two, {"three"}])

Elements can be inserted using `MapSet.put/2`:

    iex> MapSet.new([2]) |> MapSet.put(4) |> MapSet.put(0)
    MapSet.new([0, 2, 4])

By definition, sets can't contain duplicate elements: when
inserting an element in a set where it's already present, the insertion is
simply a no-op.

    iex> map_set = MapSet.new()
    iex> MapSet.put(map_set, "foo")
    MapSet.new(["foo"])
    iex> map_set |> MapSet.put("foo") |> MapSet.put("foo")
    MapSet.new(["foo"])

A `MapSet` is represented internally using the `%MapSet{}` struct. This struct
can be used whenever there's a need to pattern match on something being a `MapSet`:

    iex> match?(%MapSet{}, MapSet.new())
    true

Note that, however, the struct fields are private and must not be accessed
directly; use the functions in this module to perform operations on sets.

`MapSet`s can also be constructed starting from other collection-type data
structures: for example, see `MapSet.new/1` or `Enum.into/2`.

`MapSet` is built on top of Erlang's
[`:sets`](https://www.erlang.org/doc/man/sets.html) (version 2). This means
that they share many properties, including logarithmic time complexity. Erlang
`:sets` (version 2) are implemented on top of maps, so see the documentation
for `Map` for more information on its execution time complexity.


## Types

### internal(value)

```elixir
@opaque internal(value)
```



### t()

```elixir
@type t() :: t(term())
```



### t(value)

```elixir
@type t(value) :: %MapSet{map: internal(value)}
```



### value()

```elixir
@type value() :: term()
```



## Functions

### delete(map_set, value)

```elixir
@spec delete(t(val1), val2) :: t(val1) when val1: value(), val2: value()
```

Deletes `value` from `map_set`.

Returns a new set which is a copy of `map_set` but without `value`.

#### Examples

    iex> map_set = MapSet.new([1, 2, 3])
    iex> MapSet.delete(map_set, 4)
    MapSet.new([1, 2, 3])
    iex> MapSet.delete(map_set, 2)
    MapSet.new([1, 3])


### difference(map_set1, map_set2)

```elixir
@spec difference(t(val1), t(val2)) :: t(val1) when val1: value(), val2: value()
```

Returns a set that is `map_set1` without the members of `map_set2`.

#### Examples

    iex> MapSet.difference(MapSet.new([1, 2]), MapSet.new([2, 3, 4]))
    MapSet.new([1])


### disjoint?(map_set1, map_set2)

```elixir
@spec disjoint?(t(), t()) :: boolean()
```

Checks if `map_set1` and `map_set2` have no members in common.

#### Examples

    iex> MapSet.disjoint?(MapSet.new([1, 2]), MapSet.new([3, 4]))
    true
    iex> MapSet.disjoint?(MapSet.new([1, 2]), MapSet.new([2, 3]))
    false


### equal?(map_set1, map_set2)

```elixir
@spec equal?(t(), t()) :: boolean()
```

Checks if two sets are equal.

The comparison between elements is done using `===/2`,
which a set with `1` is not equivalent to a set with
`1.0`.

#### Examples

    iex> MapSet.equal?(MapSet.new([1, 2]), MapSet.new([2, 1, 1]))
    true
    iex> MapSet.equal?(MapSet.new([1, 2]), MapSet.new([3, 4]))
    false
    iex> MapSet.equal?(MapSet.new([1]), MapSet.new([1.0]))
    false


### filter(map_set, fun)
*(since 1.14.0)* 
```elixir
@spec filter(t(a), (a -&gt; as_boolean(term()))) :: t(a) when a: value()
```

Filters the set by returning only the elements from `map_set` for which invoking
`fun` returns a truthy value.

Also see `reject/2` which discards all elements where the function returns
a truthy value.

> #### Performance considerations {: .tip}
> 
> If you find yourself doing multiple calls to `MapSet.filter/2`
> and `MapSet.reject/2` in a pipeline, it is likely more efficient
> to use `Enum.map/2` and `Enum.filter/2` instead and convert to
> a map at the end using `MapSet.new/1`.

#### Examples

    iex> MapSet.filter(MapSet.new(1..5), fn x -> x > 3 end)
    MapSet.new([4, 5])
    
    iex> MapSet.filter(MapSet.new(["a", :b, "c"]), &is_atom/1)
    MapSet.new([:b])


### intersection(map_set1, map_set2)

```elixir
@spec intersection(t(val), t(val)) :: t(val) when val: value()
```

Returns a set containing only members that `map_set1` and `map_set2` have in common.

#### Examples

    iex> MapSet.intersection(MapSet.new([1, 2]), MapSet.new([2, 3, 4]))
    MapSet.new([2])
    
    iex> MapSet.intersection(MapSet.new([1, 2]), MapSet.new([3, 4]))
    MapSet.new([])


### member?(map_set, value)

```elixir
@spec member?(t(), value()) :: boolean()
```

Checks if `map_set` contains `value`.

#### Examples

    iex> MapSet.member?(MapSet.new([1, 2, 3]), 2)
    true
    iex> MapSet.member?(MapSet.new([1, 2, 3]), 4)
    false


### new()

```elixir
@spec new() :: t()
```

Returns a new set.

#### Examples

    iex> MapSet.new()
    MapSet.new([])


### new(enumerable)

```elixir
@spec new(Enumerable.t()) :: t()
```

Creates a set from an enumerable.

#### Examples

    iex> MapSet.new([:b, :a, 3])
    MapSet.new([3, :a, :b])
    iex> MapSet.new([3, 3, 3, 2, 2, 1])
    MapSet.new([1, 2, 3])


### new(enumerable, transform)

```elixir
@spec new(Enumerable.t(), (term() -&gt; val)) :: t(val) when val: value()
```

Creates a set from an enumerable via the transformation function.

#### Examples

    iex> MapSet.new([1, 2, 1], fn x -> 2 * x end)
    MapSet.new([2, 4])


### put(map_set, value)

```elixir
@spec put(t(val), new_val) :: t(val | new_val) when val: value(), new_val: value()
```

Inserts `value` into `map_set` if `map_set` doesn't already contain it.

#### Examples

    iex> MapSet.put(MapSet.new([1, 2, 3]), 3)
    MapSet.new([1, 2, 3])
    iex> MapSet.put(MapSet.new([1, 2, 3]), 4)
    MapSet.new([1, 2, 3, 4])


### reject(map_set, fun)
*(since 1.14.0)* 
```elixir
@spec reject(t(a), (a -&gt; as_boolean(term()))) :: t(a) when a: value()
```

Returns a set by excluding the elements from `map_set` for which invoking `fun`
returns a truthy value.

See also `filter/2`.

#### Examples

    iex> MapSet.reject(MapSet.new(1..5), fn x -> rem(x, 2) != 0 end)
    MapSet.new([2, 4])
    
    iex> MapSet.reject(MapSet.new(["a", :b, "c"]), &is_atom/1)
    MapSet.new(["a", "c"])


### size(map_set)

```elixir
@spec size(t()) :: non_neg_integer()
```

Returns the number of elements in `map_set`.

#### Examples

    iex> MapSet.size(MapSet.new([1, 2, 3]))
    3


### split_with(map_set, fun)
*(since 1.15.0)* 
```elixir
@spec split_with(t(), (term() -&gt; as_boolean(term()))) :: {t(), t()}
```

Splits the `map_set` into two `MapSet`s according to the given function `fun`.

`fun` receives each element in the `map_set` as its only argument. Returns
a tuple with the first `MapSet` containing all the elements in `map_set` for which
applying `fun` returned a truthy value, and a second `MapSet` with all the elements
for which applying `fun` returned a falsy value (`false` or `nil`).

#### Examples

    iex> {while_true, while_false} = MapSet.split_with(MapSet.new([1, 2, 3, 4]), fn v -> rem(v, 2) == 0 end)
    iex> while_true
    MapSet.new([2, 4])
    iex> while_false
    MapSet.new([1, 3])
    
    iex> {while_true, while_false} = MapSet.split_with(MapSet.new(), fn {_k, v} -> v > 50 end)
    iex> while_true
    MapSet.new([])
    iex> while_false
    MapSet.new([])


### subset?(map_set1, map_set2)

```elixir
@spec subset?(t(), t()) :: boolean()
```

Checks if `map_set1`'s members are all contained in `map_set2`.

This function checks if `map_set1` is a subset of `map_set2`.

#### Examples

    iex> MapSet.subset?(MapSet.new([1, 2]), MapSet.new([1, 2, 3]))
    true
    iex> MapSet.subset?(MapSet.new([1, 2, 3]), MapSet.new([1, 2]))
    false


### symmetric_difference(map_set1, map_set2)
*(since 1.14.0)* 
```elixir
@spec symmetric_difference(t(val1), t(val2)) :: t(val1 | val2)
when val1: value(), val2: value()
```

Returns a set with elements that are present in only one but not both sets.

#### Examples

    iex> MapSet.symmetric_difference(MapSet.new([1, 2, 3]), MapSet.new([2, 3, 4]))
    MapSet.new([1, 4])


### to_list(map_set)

```elixir
@spec to_list(t(val)) :: [val] when val: value()
```

Converts `map_set` to a list.

#### Examples

    iex> MapSet.to_list(MapSet.new([1, 2, 3]))
    [1, 2, 3]


### union(map_set1, map_set2)

```elixir
@spec union(t(val1), t(val2)) :: t(val1 | val2) when val1: value(), val2: value()
```

Returns a set containing all members of `map_set1` and `map_set2`.

#### Examples

    iex> MapSet.union(MapSet.new([1, 2]), MapSet.new([2, 3, 4]))
    MapSet.new([1, 2, 3, 4])




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
