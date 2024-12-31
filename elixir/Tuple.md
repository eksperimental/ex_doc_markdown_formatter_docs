# Tuple 
(Elixir v1.18.0-dev)

Functions for working with tuples.

Please note the following functions for tuples are found in `Kernel`:

- `elem/2` - accesses a tuple by index
- `put_elem/3` - inserts a value into a tuple by index
- `tuple_size/1` - gets the number of elements in a tuple

Tuples are intended as fixed-size containers for multiple elements.
To manipulate a collection of elements, use a list instead. `Enum`
functions do not work on tuples.

Tuples are denoted with curly braces:

    iex> {}
    {}
    iex> {1, :two, "three"}
    {1, :two, "three"}

A tuple may contain elements of different types, which are stored
contiguously in memory. Accessing any element takes constant time,
but modifying a tuple, which produces a shallow copy, takes linear time.
Tuples are good for reading data while lists are better for traversals.

Tuples are typically used either when a function has multiple return values
or for error handling. `File.read/1` returns `{:ok, contents}` if reading
the given file is successful, or else `{:error, reason}` such as when
the file does not exist.

The functions in this module that add and remove elements from tuples are
rarely used in practice, as they typically imply tuples are being used as
collections. To append to a tuple, it is preferable to extract the elements
from the old tuple with pattern matching, and then create a new tuple:

    tuple = {:ok, :example}
    
    # Avoid
    result = Tuple.insert_at(tuple, 2, %{})
    
    # Prefer
    {:ok, atom} = tuple
    result = {:ok, atom, %{}}

## Functions

### delete_at(tuple, index)

```elixir
@spec delete_at(tuple(), non_neg_integer()) :: tuple()
```

Removes an element from a tuple.

Deletes the element at the given `index` from `tuple`.
Raises an `ArgumentError` if `index` is negative or greater than
or equal to the length of `tuple`. Index is zero-based.

Inlined by the compiler.

#### Examples

    iex> tuple = {:foo, :bar, :baz}
    iex> Tuple.delete_at(tuple, 0)
    {:bar, :baz}

### duplicate(data, size)

```elixir
@spec duplicate(term(), non_neg_integer()) :: tuple()
```

Creates a new tuple.

Creates a tuple of `size` containing the
given `data` at every position.

Inlined by the compiler.

#### Examples

    iex> Tuple.duplicate(:hello, 3)
    {:hello, :hello, :hello}

### insert_at(tuple, index, value)

```elixir
@spec insert_at(tuple(), non_neg_integer(), term()) :: tuple()
```

Inserts an element into a tuple.

Inserts `value` into `tuple` at the given `index`.
Raises an `ArgumentError` if `index` is negative or greater than the
length of `tuple`. Index is zero-based.

Inlined by the compiler.

#### Examples

    iex> tuple = {:bar, :baz}
    iex> Tuple.insert_at(tuple, 0, :foo)
    {:foo, :bar, :baz}
    iex> Tuple.insert_at(tuple, 2, :bong)
    {:bar, :baz, :bong}

### product(tuple)
*(since 1.12.0)* 
```elixir
@spec product(tuple()) :: number()
```

Computes a product of tuple elements.

#### Examples

    iex> Tuple.product({255, 255})
    65025
    iex> Tuple.product({255, 1.0})
    255.0
    iex> Tuple.product({})
    1

### sum(tuple)
*(since 1.12.0)* 
```elixir
@spec sum(tuple()) :: number()
```

Computes a sum of tuple elements.

#### Examples

    iex> Tuple.sum({255, 255})
    510
    iex> Tuple.sum({255, 0.0})
    255.0
    iex> Tuple.sum({})
    0

### to_list(tuple)

```elixir
@spec to_list(tuple()) :: list()
```

Converts a tuple to a list.

Returns a new list with all the tuple elements.

Inlined by the compiler.

#### Examples

    iex> tuple = {:foo, :bar, :baz}
    iex> Tuple.to_list(tuple)
    [:foo, :bar, :baz]



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
