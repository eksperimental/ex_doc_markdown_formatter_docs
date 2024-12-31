# Enumerable protocol
(Elixir v1.18.0-dev)

Enumerable protocol used by `Enum` and `Stream` modules.

When you invoke a function in the `Enum` module, the first argument
is usually a collection that must implement this protocol.
For example, the expression `Enum.map([1, 2, 3], &(&1 * 2))`
invokes `Enumerable.reduce/3` to perform the reducing operation that
builds a mapped list by calling the mapping function `&(&1 * 2)` on
every element in the collection and consuming the element with an
accumulated list.

Internally, `Enum.map/2` is implemented as follows:

    def map(enumerable, fun) do
      reducer = fn x, acc -> {:cont, [fun.(x) | acc]} end
      Enumerable.reduce(enumerable, {:cont, []}, reducer) |> elem(1) |> :lists.reverse()
    end

Note that the user-supplied function is wrapped into a `t:reducer/0` function.
The `t:reducer/0` function must return a tagged tuple after each step,
as described in the `t:acc/0` type. At the end, `Enumerable.reduce/3`
returns `t:result/0`.

This protocol uses tagged tuples to exchange information between the
reducer function and the data type that implements the protocol. This
allows enumeration of resources, such as files, to be done efficiently
while also guaranteeing the resource will be closed at the end of the
enumeration. This protocol also allows suspension of the enumeration,
which is useful when interleaving between many enumerables is required
(as in the `zip/1` and `zip/2` functions).

This protocol requires four functions to be implemented, `reduce/3`,
`count/1`, `member?/2`, and `slice/1`. The core of the protocol is the
`reduce/3` function. All other functions exist as optimizations paths
for data structures that can implement certain properties in better
than linear time.

## Types

### acc()

```elixir
@type acc() :: {:cont, term()} | {:halt, term()} | {:suspend, term()}
```

The accumulator value for each step.

It must be a tagged tuple with one of the following "tags":

- `:cont`    - the enumeration should continue
- `:halt`    - the enumeration should halt immediately
- `:suspend` - the enumeration should be suspended immediately

Depending on the accumulator value, the result returned by
`Enumerable.reduce/3` will change. Please check the `t:result/0`
type documentation for more information.

In case a `t:reducer/0` function returns a `:suspend` accumulator,
it must be explicitly handled by the caller and never leak.

### continuation()

```elixir
@type continuation() :: (acc() -&gt; result())
```

A partially applied reduce function.

The continuation is the closure returned as a result when
the enumeration is suspended. When invoked, it expects
a new accumulator and it returns the result.

A continuation can be trivially implemented as long as the reduce
function is defined in a tail recursive fashion. If the function
is tail recursive, all the state is passed as arguments, so
the continuation is the reducing function partially applied.

### reducer()

```elixir
@type reducer() :: (element :: term(), element_acc :: term() -&gt; acc())
```

The reducer function.

Should be called with the `enumerable` element and the
accumulator contents.

Returns the accumulator for the next enumeration step.

### result()

```elixir
@type result() ::
  {:done, term()} | {:halted, term()} | {:suspended, term(), continuation()}
```

The result of the reduce operation.

It may be *done* when the enumeration is finished by reaching
its end, or *halted*/*suspended* when the enumeration was halted
or suspended by the tagged accumulator.

In case the tagged `:halt` accumulator is given, the `:halted` tuple
with the accumulator must be returned. Functions like `Enum.take_while/2`
use `:halt` underneath and can be used to test halting enumerables.

In case the tagged `:suspend` accumulator is given, the caller must
return the `:suspended` tuple with the accumulator and a continuation.
The caller is then responsible of managing the continuation and the
caller must always call the continuation, eventually halting or continuing
until the end. `Enum.zip/2` uses suspension, so it can be used to test
whether your implementation handles suspension correctly. You can also use
`Stream.zip/2` with `Enum.take_while/2` to test the combination of
`:suspend` with `:halt`.

### slicing_fun()

```elixir
@type slicing_fun() :: (start :: non_neg_integer(),
                  length :: pos_integer(),
                  step :: pos_integer() -&gt;
                    [term()])
```

A slicing function that receives the initial position,
the number of elements in the slice, and the step.

The `start` position is a number `>= 0` and guaranteed to
exist in the `enumerable`. The length is a number `>= 1`
in a way that `start + length * step <= count`, where
`count` is the maximum amount of elements in the enumerable.

The function should return a non empty list where
the amount of elements is equal to `length`.

### t()

```elixir
@type t() :: term()
```

All the types that implement this protocol.

### t(_element)
*(since 1.14.0)* 
```elixir
@type t(_element) :: t()
```

An enumerable of elements of type `element`.

This type is equivalent to `t:t/0` but is especially useful for documentation.

For example, imagine you define a function that expects an enumerable of
integers and returns an enumerable of strings:

    @spec integers_to_strings(Enumerable.t(integer())) :: Enumerable.t(String.t())
    def integers_to_strings(integers) do
      Stream.map(integers, &Integer.to_string/1)
    end

### to_list_fun()

```elixir
@type to_list_fun() :: (t() -&gt; [term()])
```

Receives an enumerable and returns a list.

## Functions

### count(enumerable)

```elixir
@spec count(t()) :: {:ok, non_neg_integer()} | {:error, module()}
```

Retrieves the number of elements in the `enumerable`.

It should return `{:ok, count}` if you can count the number of elements
in `enumerable` in a faster way than fully traversing it.

Otherwise it should return `{:error, __MODULE__}` and a default algorithm
built on top of `reduce/3` that runs in linear time will be used.

### member?(enumerable, element)

```elixir
@spec member?(t(), term()) :: {:ok, boolean()} | {:error, module()}
```

Checks if an `element` exists within the `enumerable`.

It should return `{:ok, boolean}` if you can check the membership of a
given element in `enumerable` with `===/2` without traversing the whole
of it.

Otherwise it should return `{:error, __MODULE__}` and a default algorithm
built on top of `reduce/3` that runs in linear time will be used.

When called outside guards, the [`in`](\`in/2\`) and [`not in`](\`in/2\`)
operators work by using this function.

### reduce(enumerable, acc, fun)

```elixir
@spec reduce(t(), acc(), reducer()) :: result()
```

Reduces the `enumerable` into an element.

Most of the operations in `Enum` are implemented in terms of reduce.
This function should apply the given `t:reducer/0` function to each
element in the `enumerable` and proceed as expected by the returned
accumulator.

See the documentation of the types `t:result/0` and `t:acc/0` for
more information.

#### Examples

As an example, here is the implementation of `reduce` for lists:

    def reduce(_list, {:halt, acc}, _fun), do: {:halted, acc}
    def reduce(list, {:suspend, acc}, fun), do: {:suspended, acc, &reduce(list, &1, fun)}
    def reduce([], {:cont, acc}, _fun), do: {:done, acc}
    def reduce([head | tail], {:cont, acc}, fun), do: reduce(tail, fun.(head, acc), fun)

### slice(enumerable)

```elixir
@spec slice(t()) ::
  {:ok, size :: non_neg_integer(), slicing_fun() | to_list_fun()}
  | {:error, module()}
```

Returns a function that slices the data structure contiguously.

It should return either:

- `{:ok, size, slicing_fun}` - if the `enumerable` has a known
  bound and can access a position in the `enumerable` without
  traversing all previous elements. The `slicing_fun` will receive
  a `start` position, the `amount` of elements to fetch, and a
  `step`.

- `{:ok, size, to_list_fun}` - if the `enumerable` has a known bound
  and can access a position in the `enumerable` by first converting
  it to a list via `to_list_fun`.

- `{:error, __MODULE__}` - the enumerable cannot be sliced efficiently
  and a default algorithm built on top of `reduce/3` that runs in
  linear time will be used.

#### Differences to `count/1`

The `size` value returned by this function is used for boundary checks,
therefore it is extremely important that this function only returns `:ok`
if retrieving the `size` of the `enumerable` is cheap, fast, and takes
constant time. Otherwise the simplest of operations, such as
`Enum.at(enumerable, 0)`, will become too expensive.

On the other hand, the `count/1` function in this protocol should be
implemented whenever you can count the number of elements in the collection
without traversing it.



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
