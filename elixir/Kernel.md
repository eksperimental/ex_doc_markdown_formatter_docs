# Kernel 
(Elixir v1.18.0-dev)

`Kernel` is Elixir's default environment.

It mainly consists of:

- basic language primitives, such as arithmetic operators, spawning of processes,
  data type handling, and others
- macros for control-flow and defining new functionality (modules, functions, and the like)
- guard checks for augmenting pattern matching

You can invoke `Kernel` functions and macros anywhere in Elixir code
without the use of the `Kernel.` prefix since they have all been
automatically imported. For example, in IEx, you can call:

    iex> is_number(13)
    true

If you don't want to import a function or macro from `Kernel`, use the `:except`
option and then list the function/macro by arity:

    import Kernel, except: [if: 2, unless: 2]

See `import/2` for more information on importing.

Elixir also has special forms that are always imported and
cannot be skipped. These are described in `Kernel.SpecialForms`.

## The standard library

`Kernel` provides the basic capabilities the Elixir standard library
is built on top of. It is recommended to explore the standard library
for advanced functionality. Here are the main groups of modules in the
standard library (this list is not a complete reference, see the
documentation sidebar for all entries).

### Built-in types

The following modules handle Elixir built-in data types:

- `Atom` - literal constants with a name (`true`, `false`, and `nil` are atoms)
- `Float` - numbers with floating point precision
- `Function` - a reference to code chunk, created with the `fn/1` special form
- `Integer` - whole numbers (not fractions)
- `List` - collections of a variable number of elements (linked lists)
- `Map` - collections of key-value pairs
- `Process` - light-weight threads of execution
- `Port` - mechanisms to interact with the external world
- `Tuple` - collections of a fixed number of elements

There are two data types without an accompanying module:

- Bitstring - a sequence of bits, created with `<<>>/1`.
  When the number of bits is divisible by 8, they are called binaries and can
  be manipulated with Erlang's `:binary` module
- Reference - a unique value in the runtime system, created with `make_ref/0`

### Data types

Elixir also provides other data types that are built on top of the types
listed above. Some of them are:

- `Date` - `year-month-day` structs in a given calendar
- `DateTime` - date and time with time zone in a given calendar
- `Exception` - data raised from errors and unexpected scenarios
- `MapSet` - unordered collections of unique elements
- `NaiveDateTime` - date and time without time zone in a given calendar
- `Keyword` - lists of two-element tuples, often representing optional values
- `Range` - inclusive ranges between two integers
- `Regex` - regular expressions
- `String` - UTF-8 encoded binaries representing characters
- `Time` - `hour:minute:second` structs in a given calendar
- `URI` - representation of URIs that identify resources
- `Version` - representation of versions and requirements

### System modules

Modules that interface with the underlying system, such as:

- `IO` - handles input and output
- `File` - interacts with the underlying file system
- `Path` - manipulates file system paths
- `System` - reads and writes system information

### Protocols

Protocols add polymorphic dispatch to Elixir. They are contracts
implementable by data types. See `Protocol` for more information on
protocols. Elixir provides the following protocols in the standard library:

- `Collectable` - collects data into a data type
- `Enumerable` - handles collections in Elixir. The `Enum` module
  provides eager functions for working with collections, the `Stream`
  module provides lazy functions
- `Inspect` - converts data types into their programming language
  representation
- `List.Chars` - converts data types to their outside world
  representation as charlists (non-programming based)
- `String.Chars` - converts data types to their outside world
  representation as strings (non-programming based)

### Process-based and application-centric functionality

The following modules build on top of processes to provide concurrency,
fault-tolerance, and more.

- `Agent` - a process that encapsulates mutable state
- `Application` - functions for starting, stopping and configuring
  applications
- `GenServer` - a generic client-server API
- `Registry` - a key-value process-based storage
- `Supervisor` - a process that is responsible for starting,
  supervising and shutting down other processes
- `Task` - a process that performs computations
- `Task.Supervisor` - a supervisor for managing tasks exclusively

### Supporting documents

Under the "Pages" section in sidebar you will find tutorials, guides,
and reference documents that outline Elixir semantics and behaviors
in more detail. Those are:

- [Compatibility and deprecations](compatibility-and-deprecations.md) - lists
  compatibility between every Elixir version and Erlang/OTP, release schema;
  lists all deprecated functions, when they were deprecated and alternatives
- [Library guidelines](library-guidelines.md) - general guidelines, anti-patterns,
  and rules for those writing libraries
- [Naming conventions](naming-conventions.md) - naming conventions for Elixir code
- [Operators reference](operators.md) - lists all Elixir operators and their precedences
- [Patterns and guards](patterns-and-guards.md) - an introduction to patterns,
  guards, and extensions
- [Syntax reference](syntax-reference.md) - the language syntax reference
- [Typespecs reference](typespecs.md)- types and function specifications, including list of types
- [Unicode syntax](unicode-syntax.md) - outlines Elixir support for Unicode

## Guards

This module includes the built-in guards used by Elixir developers.
They are a predefined set of functions and macros that augment pattern
matching, typically invoked after the `when` operator. For example:

    def drive(%User{age: age}) when age >= 16 do
      ...
    end

The clause above will only be invoked if the user's age is more than
or equal to 16. Guards also support joining multiple conditions with
`and` and `or`. The whole guard is true if all guard expressions will
evaluate to `true`. A more complete introduction to guards is available
in the [Patterns and guards](patterns-and-guards.md) page.

## Truthy and falsy values

Besides the booleans `true` and `false`, Elixir has the
concept of a "truthy" or "falsy" value.

- a value is truthy when it is neither `false` nor `nil`
- a value is falsy when it is either `false` or `nil`

Elixir has functions, like `and/2`, that *only* work with
booleans, but also functions that work with these
truthy/falsy values, like `&&/2` and `!/1`.

## Structural comparison

The functions in this module perform structural comparison. This allows
different data types to be compared using comparison operators:

    1 < :an_atom

This is possible so Elixir developers can create collections, such as
dictionaries and ordered sets, that store a mixture of data types in them.
To understand why this matters, let's discuss the two types of comparisons
we find in software: *structural* and *semantic*.

Structural means we are comparing the underlying data structures and we often
want those operations to be as fast as possible, because it is used to power
several algorithms and data structures in the language. A semantic comparison
worries about what each data type represents. For example, semantically
speaking, it doesn't make sense to compare `Time` with `Date`.

One example that shows the differences between structural and semantic
comparisons are strings: "alien" sorts less than "office" (`"alien" < "office"`)
but "álien" is greater than "office". This happens because `<` compares the
underlying bytes that form the string. If you were doing alphabetical listing,
you may want "álien" to also appear before "office".

This means **comparisons in Elixir are structural**, as it has the goal
of comparing data types as efficiently as possible to create flexible
and performant data structures. This distinction is specially important
for functions that provide ordering, such as `>/2`, `</2`, `>=/2`,
`<=/2`, `min/2`, and `max/2`. For example:

    ~D[2017-03-31] > ~D[2017-04-01]

will return `true` because structural comparison compares the `:day`
field before `:month` or `:year`. Luckily, the Elixir compiler will
detect whenever comparing structs or whenever comparing code that is
either always true or false, and emit a warning accordingly.

In order to perform semantic comparisons, the relevant data-types
provide a `compare/2` function, such as `Date.compare/2`:

    iex> Date.compare(~D[2017-03-31], ~D[2017-04-01])
    :lt

Alternatively, you can use the functions in the `Enum` module to
sort or compute a maximum/minimum:

    iex> Enum.sort([~D[2017-03-31], ~D[2017-04-01]], Date)
    [~D[2017-03-31], ~D[2017-04-01]]
    iex> Enum.max([~D[2017-03-31], ~D[2017-04-01]], Date)
    ~D[2017-04-01]

The second argument is precisely the module to be used for semantic
comparison. Keeping this distinction is important, because if semantic
comparison was used by default for implementing data structures and
algorithms, they could become orders of magnitude slower\!

Finally, note there is an overall structural sorting order, called
"Term Ordering", defined below. This order is provided for reference
purposes, it is not required by Elixir developers to know it by heart.

### Term ordering

    number < atom < reference < function < port < pid < tuple < map < list < bitstring

When comparing two numbers of different types (a number being either
an integer or a float), a conversion to the type with greater precision
will always occur, unless the comparison operator used is either `===/2`
or `!==/2`. A float will be considered more precise than an integer, unless
the float is greater/less than +/-9007199254740992.0 respectively,
at which point all the significant figures of the float are to the left
of the decimal point. This behavior exists so that the comparison of large
numbers remains transitive.

The collection types are compared using the following rules:

- Tuples are compared by size, then element by element.
- Maps are compared by size, then by key-value pairs.
- Lists are compared element by element.
- Bitstrings are compared byte by byte, incomplete bytes are compared bit by bit.
- Atoms are compared using their string value, codepoint by codepoint.

### Examples

We can check the truthiness of a value by using the `!/1`
function twice.

Truthy values:

    iex> !!true
    true
    iex> !!5
    true
    iex> !![1,2]
    true
    iex> !!"foo"
    true

Falsy values (of which there are exactly two):

    iex> !!false
    false
    iex> !!nil
    false

## Inlining

Some of the functions described in this module are inlined by
the Elixir compiler into their Erlang counterparts in the
[`:erlang`](\`:erlang\`) module.
Those functions are called BIFs (built-in internal functions)
in Erlang-land and they exhibit interesting properties, as some
of them are allowed in guards and others are used for compiler
optimizations.

Most of the inlined functions can be seen in effect when
capturing the function:

    iex> &Kernel.is_atom/1
    &:erlang.is_atom/1

Those functions will be explicitly marked in their docs as
"inlined by the compiler".


## Guards

### left * right

```elixir
@spec integer() * integer() :: integer()
@spec float() * float() :: float()
@spec integer() * float() :: float()
@spec float() * integer() :: float()
```

Arithmetic multiplication operator.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 * 2
    2


### +value

```elixir
@spec +integer() :: integer()
@spec +float() :: float()
```

Arithmetic positive unary operator.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> +1
    1


### left + right

```elixir
@spec integer() + integer() :: integer()
@spec float() + float() :: float()
@spec integer() + float() :: float()
@spec float() + integer() :: float()
```

Arithmetic addition operator.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 + 2
    3


### -value

```elixir
@spec -0 :: 0
@spec -pos_integer() :: neg_integer()
@spec -neg_integer() :: pos_integer()
@spec -float() :: float()
```

Arithmetic negative unary operator.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> -2
    -2


### left - right

```elixir
@spec integer() - integer() :: integer()
@spec float() - float() :: float()
@spec integer() - float() :: float()
@spec float() - integer() :: float()
```

Arithmetic subtraction operator.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 - 2
    -1


### left / right

```elixir
@spec number() / number() :: float()
```

Arithmetic division operator.

The result is always a float. Use `div/2` and `rem/2` if you want
an integer division or the remainder.

Raises `ArithmeticError` if `right` is 0 or 0.0.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    1 / 2
    #=> 0.5
    
    -3.0 / 2.0
    #=> -1.5
    
    5 / 1
    #=> 5.0
    
    7 / 0
    ** (ArithmeticError) bad argument in arithmetic expression


### left != right

```elixir
@spec term() != term() :: boolean()
```

Not equal to operator.

Returns `true` if the two terms are not equal.

This operator considers 1 and 1.0 to be equal. For match
comparison, use `!==/2` instead.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 != 2
    true
    
    iex> 1 != 1.0
    false


### left !== right

```elixir
@spec term() !== term() :: boolean()
```

Strictly not equal to operator.

Returns `true` if the two terms are not exactly equal.
See `===/2` for a definition of what is considered "exactly equal".

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 !== 2
    true
    
    iex> 1 !== 1.0
    true


### left &lt; right

```elixir
@spec term() &lt; term() :: boolean()
```

Less-than operator.

Returns `true` if `left` is less than `right`.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 < 2
    true


### left &lt;= right

```elixir
@spec term() &lt;= term() :: boolean()
```

Less-than or equal to operator.

Returns `true` if `left` is less than or equal to `right`.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 <= 2
    true


### left == right

```elixir
@spec term() == term() :: boolean()
```

Equal to operator. Returns `true` if the two terms are equal.

This operator considers 1 and 1.0 to be equal. For stricter
semantics, use `===/2` instead.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 == 2
    false
    
    iex> 1 == 1.0
    true


### left === right

```elixir
@spec term() === term() :: boolean()
```

Strictly equal to operator.

Returns `true` if the two terms are exactly equal.

The terms are only considered to be exactly equal if they
have the same value and are of the same type. For example,
`1 == 1.0` returns `true`, but since they are of different
types, `1 === 1.0` returns `false`.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 === 2
    false
    
    iex> 1 === 1.0
    false


### left &gt; right

```elixir
@spec term() &gt; term() :: boolean()
```

Greater-than operator.

Returns `true` if `left` is more than `right`.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 > 2
    false


### left &gt;= right

```elixir
@spec term() &gt;= term() :: boolean()
```

Greater-than or equal to operator.

Returns `true` if `left` is more than or equal to `right`.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> 1 >= 2
    false


### abs(number)

```elixir
@spec abs(number()) :: number()
```

Returns an integer or float which is the arithmetical absolute value of `number`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> abs(-3.33)
    3.33
    
    iex> abs(-3)
    3


### left and right
*(macro)* 


Strictly boolean "and" operator.

If `left` is `false`, returns `false`, otherwise returns `right`.

Requires only the `left` operand to be a boolean since it short-circuits. If
the `left` operand is not a boolean, a `BadBooleanError` exception is raised.

Allowed in guard tests.

#### Examples

    iex> true and false
    false
    
    iex> true and "yay!"
    "yay!"
    
    iex> "yay!" and true
    ** (BadBooleanError) expected a boolean on left-side of "and", got: "yay!"


### binary_part(binary, start, size)

```elixir
@spec binary_part(binary(), non_neg_integer(), integer()) :: binary()
```

Extracts the part of the binary at `start` with `size`.

If `start` or `size` reference in any way outside the binary,
an `ArgumentError` exception is raised.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> binary_part("foo", 1, 2)
    "oo"

A negative `size` can be used to extract bytes that come *before* the byte
at `start`:

    iex> binary_part("Hello", 5, -3)
    "llo"

An `ArgumentError` is raised when the `size` is outside of the binary:

    binary_part("Hello", 0, 10)
    ** (ArgumentError) argument error


### bit_size(bitstring)

```elixir
@spec bit_size(bitstring()) :: non_neg_integer()
```

Returns an integer which is the size in bits of `bitstring`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> bit_size(<<433::16, 3::3>>)
    19
    
    iex> bit_size(<<1, 2, 3>>)
    24


### byte_size(bitstring)

```elixir
@spec byte_size(bitstring()) :: non_neg_integer()
```

Returns the number of bytes needed to contain `bitstring`.

That is, if the number of bits in `bitstring` is not divisible by 8, the
resulting number of bytes will be rounded up (by excess). This operation
happens in constant time.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> byte_size(<<433::16, 3::3>>)
    3
    
    iex> byte_size(<<1, 2, 3>>)
    3


### ceil(number)
*(since 1.8.0)* 
```elixir
@spec ceil(number()) :: integer()
```

Returns the smallest integer greater than or equal to `number`.

If you want to perform ceil operation on other decimal places,
use `Float.ceil/2` instead.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> ceil(10)
    10
    
    iex> ceil(10.1)
    11
    
    iex> ceil(-10.1)
    -10


### div(dividend, divisor)

```elixir
@spec div(integer(), neg_integer() | pos_integer()) :: integer()
```

Performs an integer division.

Raises an `ArithmeticError` exception if one of the arguments is not an
integer, or when the `divisor` is `0`.

`div/2` performs *truncated* integer division. This means that
the result is always rounded towards zero.

If you want to perform floored integer division (rounding towards negative infinity),
use `Integer.floor_div/2` instead.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    div(5, 2)
    #=> 2
    
    div(6, -4)
    #=> -1
    
    div(-99, 2)
    #=> -49
    
    div(100, 0)
    ** (ArithmeticError) bad argument in arithmetic expression


### elem(tuple, index)

```elixir
@spec elem(tuple(), non_neg_integer()) :: term()
```

Gets the element at the zero-based `index` in `tuple`.

It raises `ArgumentError` when index is negative or it is out of range of the tuple elements.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    tuple = {:foo, :bar, 3}
    elem(tuple, 1)
    #=> :bar
    
    elem({}, 0)
    ** (ArgumentError) argument error
    
    elem({:foo, :bar}, 2)
    ** (ArgumentError) argument error


### floor(number)
*(since 1.8.0)* 
```elixir
@spec floor(number()) :: integer()
```

Returns the largest integer smaller than or equal to `number`.

If you want to perform floor operation on other decimal places,
use `Float.floor/2` instead.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> floor(10)
    10
    
    iex> floor(9.7)
    9
    
    iex> floor(-9.7)
    -10


### hd(list)

```elixir
@spec hd(nonempty_maybe_improper_list(elem, term())) :: elem when elem: term()
```

Returns the head of a list. Raises `ArgumentError` if the list is empty.

The head of a list is its first element.

It works with improper lists.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    hd([1, 2, 3, 4])
    #=> 1
    
    hd([1 | 2])
    #=> 1

Giving it an empty list raises:

    hd([])
    ** (ArgumentError) argument error


### left in right
*(macro)* 


Membership operator.

Checks if the element on the left-hand side is a member of the
collection on the right-hand side.

#### Examples

    iex> x = 1
    iex> x in [1, 2, 3]
    true

This operator (which is a macro) simply translates to a call to
`Enum.member?/2`. The example above would translate to:

    Enum.member?([1, 2, 3], x)

Elixir also supports `left not in right`, which evaluates to
`not(left in right)`:

    iex> x = 1
    iex> x not in [1, 2, 3]
    false

#### Guards

The `in/2` operator (as well as `not in`) can be used in guard clauses as
long as the right-hand side is a range or a list.

If the right-hand side is a list, Elixir will expand the operator to a valid
guard expression which needs to check each value. For example:

    when x in [1, 2, 3]

translates to:

    when x === 1 or x === 2 or x === 3

However, this construct will be inefficient for large lists. In such cases, it
is best to stop using guards and use a more appropriate data structure, such
as `MapSet`.

If the right-hand side is a range, a more efficient comparison check will be
done. For example:

    when x in 1..1000

translates roughly to:

    when x >= 1 and x <= 1000

##### AST considerations

`left not in right` is parsed by the compiler into the AST:

    {:not, _, [{:in, _, [left, right]}]}

This is the same AST as `not(left in right)`.

Additionally, `Macro.to_string/2` and `Code.format_string!/2`
will translate all occurrences of this AST to `left not in right`.


### is_atom(term)

```elixir
@spec is_atom(term()) :: boolean()
```

Returns `true` if `term` is an atom, otherwise returns `false`.

Note `true`, `false`, and `nil` are atoms in Elixir, as well as
module names. Therefore this function will return `true` to all
of those values.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> is_atom(:name)
    true
    
    iex> is_atom(false)
    true
    
    iex> is_atom(AnAtom)
    true
    
    iex> is_atom("string")
    false


### is_binary(term)

```elixir
@spec is_binary(term()) :: boolean()
```

Returns `true` if `term` is a binary, otherwise returns `false`.

A binary always contains a complete number of bytes.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> is_binary("foo")
    true
    iex> is_binary(<<1::3>>)
    false


### is_bitstring(term)

```elixir
@spec is_bitstring(term()) :: boolean()
```

Returns `true` if `term` is a bitstring (including a binary), otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> is_bitstring("foo")
    true
    iex> is_bitstring(<<1::3>>)
    true


### is_boolean(term)

```elixir
@spec is_boolean(term()) :: boolean()
```

Returns `true` if `term` is either the atom `true` or the atom `false` (i.e.,
a boolean), otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> is_boolean(false)
    true
    
    iex> is_boolean(true)
    true
    
    iex> is_boolean(:test)
    false


### is_exception(term)
*(since 1.11.0)* *(macro)* 


Returns `true` if `term` is an exception, otherwise returns `false`.

Allowed in guard tests.

#### Examples

    iex> is_exception(%RuntimeError{})
    true
    
    iex> is_exception(%{})
    false


### is_exception(term, name)
*(since 1.11.0)* *(macro)* 


Returns `true` if `term` is an exception of `name`, otherwise returns `false`.

Allowed in guard tests.

#### Examples

    iex> is_exception(%RuntimeError{}, RuntimeError)
    true
    
    iex> is_exception(%RuntimeError{}, Macro.Env)
    false


### is_float(term)

```elixir
@spec is_float(term()) :: boolean()
```

Returns `true` if `term` is a floating-point number, otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.


### is_function(term)

```elixir
@spec is_function(term()) :: boolean()
```

Returns `true` if `term` is a function, otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> is_function(fn x -> x + x end)
    true
    
    iex> is_function("not a function")
    false


### is_function(term, arity)

```elixir
@spec is_function(term(), non_neg_integer()) :: boolean()
```

Returns `true` if `term` is a function that can be applied with `arity` number of arguments;
otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> is_function(fn x -> x * 2 end, 1)
    true
    iex> is_function(fn x -> x * 2 end, 2)
    false


### is_integer(term)

```elixir
@spec is_integer(term()) :: boolean()
```

Returns `true` if `term` is an integer, otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.


### is_list(term)

```elixir
@spec is_list(term()) :: boolean()
```

Returns `true` if `term` is a list with zero or more elements, otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.


### is_map(term)

```elixir
@spec is_map(term()) :: boolean()
```

Returns `true` if `term` is a map, otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.

> #### Structs are maps {: .info}
> 
> Structs are also maps, and many of Elixir data structures are implemented
> using structs: `Range`s, `Regex`es, `Date`s...
> 
>     iex> is_map(1..10)
>     true
>     iex> is_map(~D[2024-04-18])
>     true

> If you mean to specifically check for non-struct maps, use
> `is_non_struct_map/1` instead.
> 
>     iex> is_non_struct_map(1..10)
>     false


### is_map_key(map, key)
*(since 1.10.0)* 
```elixir
@spec is_map_key(map(), term()) :: boolean()
```

Returns `true` if `key` is a key in `map`, otherwise returns `false`.

It raises `BadMapError` if the first element is not a map.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> is_map_key(%{a: "foo", b: "bar"}, :a)
    true
    
    iex> is_map_key(%{a: "foo", b: "bar"}, :c)
    false


### is_nil(term)
*(macro)* 


Returns `true` if `term` is `nil`, `false` otherwise.

Allowed in guard clauses.

#### Examples

    iex> is_nil(1 + 2)
    false
    
    iex> is_nil(nil)
    true


### is_non_struct_map(term)
*(since 1.17.0)* *(macro)* 


Returns `true` if `term` is a map that is not a struct, otherwise
returns `false`.

Allowed in guard tests.

#### Examples

    iex> is_non_struct_map(%{})
    true
    
    iex> is_non_struct_map(URI.parse("/"))
    false
    
    iex> is_non_struct_map(nil)
    false


### is_number(term)

```elixir
@spec is_number(term()) :: boolean()
```

Returns `true` if `term` is either an integer or a floating-point number;
otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.


### is_pid(term)

```elixir
@spec is_pid(term()) :: boolean()
```

Returns `true` if `term` is a PID (process identifier), otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.


### is_port(term)

```elixir
@spec is_port(term()) :: boolean()
```

Returns `true` if `term` is a port identifier, otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.


### is_reference(term)

```elixir
@spec is_reference(term()) :: boolean()
```

Returns `true` if `term` is a reference, otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.


### is_struct(term)
*(since 1.10.0)* *(macro)* 


Returns `true` if `term` is a struct, otherwise returns `false`.

Allowed in guard tests.

#### Examples

    iex> is_struct(URI.parse("/"))
    true
    
    iex> is_struct(%{})
    false


### is_struct(term, name)
*(since 1.11.0)* *(macro)* 


Returns `true` if `term` is a struct of `name`, otherwise returns `false`.

`is_struct/2` does not check that `name` exists and is a valid struct.
If you want such validations, you must pattern match on the struct
instead, such as `match?(%URI{}, arg)`.

Allowed in guard tests.

#### Examples

    iex> is_struct(URI.parse("/"), URI)
    true
    
    iex> is_struct(URI.parse("/"), Macro.Env)
    false


### is_tuple(term)

```elixir
@spec is_tuple(term()) :: boolean()
```

Returns `true` if `term` is a tuple, otherwise returns `false`.

Allowed in guard tests. Inlined by the compiler.


### length(list)

```elixir
@spec length(list()) :: non_neg_integer()
```

Returns the length of `list`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> length([1, 2, 3, 4, 5, 6, 7, 8, 9])
    9


### map_size(map)

```elixir
@spec map_size(map()) :: non_neg_integer()
```

Returns the size of a map.

The size of a map is the number of key-value pairs that the map contains.

This operation happens in constant time.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> map_size(%{a: "foo", b: "bar"})
    2


### node()

```elixir
@spec node() :: node()
```

Returns an atom representing the name of the local node.
If the node is not alive, `:nonode@nohost` is returned instead.

Allowed in guard tests. Inlined by the compiler.


### node(arg)

```elixir
@spec node(pid() | reference() | port()) :: node()
```

Returns the node where the given argument is located.
The argument can be a PID, a reference, or a port.
If the local node is not alive, `:nonode@nohost` is returned.

Allowed in guard tests. Inlined by the compiler.


### not value

```elixir
@spec not true :: false
@spec not false :: true
```

Strictly boolean "not" operator.

`value` must be a boolean; if it's not, an `ArgumentError` exception is raised.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> not false
    true


### left or right
*(macro)* 


Strictly boolean "or" operator.

If `left` is `true`, returns `true`, otherwise returns `right`.

Requires only the `left` operand to be a boolean since it short-circuits.
If the `left` operand is not a boolean, a `BadBooleanError` exception is
raised.

Allowed in guard tests.

#### Examples

    iex> true or false
    true
    
    iex> false or 42
    42
    
    iex> 42 or false
    ** (BadBooleanError) expected a boolean on left-side of "or", got: 42


### rem(dividend, divisor)

```elixir
@spec rem(integer(), neg_integer() | pos_integer()) :: integer()
```

Computes the remainder of an integer division.

`rem/2` uses truncated division, which means that
the result will always have the sign of the `dividend`.

Raises an `ArithmeticError` exception if one of the arguments is not an
integer, or when the `divisor` is `0`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> rem(5, 2)
    1
    iex> rem(6, -4)
    2


### round(number)

```elixir
@spec round(number()) :: integer()
```

Rounds a number to the nearest integer.

If the number is equidistant to the two nearest integers, rounds away from zero.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> round(5.6)
    6
    
    iex> round(5.2)
    5
    
    iex> round(-9.9)
    -10
    
    iex> round(-9)
    -9
    
    iex> round(2.5)
    3
    
    iex> round(-2.5)
    -3


### self()

```elixir
@spec self() :: pid()
```

Returns the PID (process identifier) of the calling process.

Allowed in guard clauses. Inlined by the compiler.


### tl(list)

```elixir
@spec tl(nonempty_maybe_improper_list(elem, last)) ::
  maybe_improper_list(elem, last) | last
when elem: term(), last: term()
```

Returns the tail of a list. Raises `ArgumentError` if the list is empty.

The tail of a list is the list without its first element.

It works with improper lists.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    tl([1, 2, 3, :go])
    #=> [2, 3, :go]
    
    tl([:one])
    #=> []
    
    tl([:a, :b | :improper_end])
    #=> [:b | :improper_end]
    
    tl([:a | %{b: 1}])
    #=> %{b: 1}

Giving it an empty list raises:

    tl([])
    ** (ArgumentError) argument error


### trunc(number)

```elixir
@spec trunc(number()) :: integer()
```

Returns the integer part of `number`.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> trunc(5.4)
    5
    
    iex> trunc(-5.99)
    -5
    
    iex> trunc(-5)
    -5


### tuple_size(tuple)

```elixir
@spec tuple_size(tuple()) :: non_neg_integer()
```

Returns the size of a tuple.

This operation happens in constant time.

Allowed in guard tests. Inlined by the compiler.

#### Examples

    iex> tuple_size({:a, :b, :c})
    3


## Functions

### left &amp;&amp; right
*(macro)* 


Boolean "and" operator.

Provides a short-circuit operator that evaluates and returns
the second expression only if the first one evaluates to a truthy value
(neither `false` nor `nil`). Returns the first expression
otherwise.

Not allowed in guard clauses.

#### Examples

    iex> Enum.empty?([]) && Enum.empty?([])
    true
    
    iex> List.first([]) && true
    nil
    
    iex> Enum.empty?([]) && List.first([1])
    1
    
    iex> false && throw(:bad)
    false

Note that, unlike `and/2`, this operator accepts any expression
as the first argument, not only booleans.


### base ** exponent
*(since 1.13.0)* 
```elixir
@spec integer() ** non_neg_integer() :: integer()
@spec integer() ** neg_integer() :: float()
@spec float() ** float() :: float()
@spec integer() ** float() :: float()
@spec float() ** integer() :: float()
```

Power operator.

It takes two numbers for input. If both are integers and the right-hand
side (the `exponent`) is also greater than or equal to 0, then the result
will also be an integer. Otherwise it returns a float.

#### Examples

    iex> 2 ** 2
    4
    iex> 2 ** -4
    0.0625
    
    iex> 2.0 ** 2
    4.0
    iex> 2 ** 2.0
    4.0


### left ++ right

```elixir
@spec [] ++ a :: a when a: term()
@spec [...] ++ term() :: maybe_improper_list()
```

List concatenation operator. Concatenates a proper list and a term, returning a list.

The complexity of `a ++ b` is proportional to `length(a)`, so avoid repeatedly
appending to lists of arbitrary length, for example, `list ++ [element]`.
Instead, consider prepending via `[element | rest]` and then reversing.

If the `right` operand is not a proper list, it returns an improper list.
If the `left` operand is not a proper list, it raises `ArgumentError`.
If the `left` operand is an empty list, it returns the `right` operand.

Inlined by the compiler.

#### Examples

    iex> [1] ++ [2, 3]
    [1, 2, 3]
    
    iex> ~c"foo" ++ ~c"bar"
    ~c"foobar"
    
    # a non-list on the right will return an improper list
    # with said element at the end
    iex> [1, 2] ++ 3
    [1, 2 | 3]
    iex> [1, 2] ++ {3, 4}
    [1, 2 | {3, 4}]
    
    # improper list on the right will return an improper list
    iex> [1] ++ [2 | 3]
    [1, 2 | 3]
    
    # empty list on the left will return the right operand
    iex> [] ++ 1
    1

The `++/2` operator is right associative, meaning:

    iex> [1, 2, 3] -- [1] ++ [2]
    [3]

As it is equivalent to:

    iex> [1, 2, 3] -- ([1] ++ [2])
    [3]


### left -- right

```elixir
@spec list() -- list() :: list()
```

List subtraction operator. Removes the first occurrence of an element
on the left list for each element on the right.

This function is optimized so the complexity of `a -- b` is proportional
to `length(a) * log(length(b))`. See also the [Erlang efficiency
guide](https://www.erlang.org/doc/system/efficiency_guide.html).

Inlined by the compiler.

#### Examples

    iex> [1, 2, 3] -- [1, 2]
    [3]
    
    iex> [1, 2, 3, 2, 1] -- [1, 2, 2]
    [3, 1]

The `--/2` operator is right associative, meaning:

    iex> [1, 2, 3] -- [2] -- [3]
    [1, 3]

As it is equivalent to:

    iex> [1, 2, 3] -- ([2] -- [3])
    [1, 3]


### ..
*(since 1.14.0)* *(macro)* 


Creates the full-slice range `0..-1//1`.

This macro returns a range with the following properties:

- When enumerated, it is empty

- When used as a `slice`, it returns the sliced element as is

See `..///3` and the `Range` module for more information.

#### Examples

    iex> Enum.to_list(..)
    []
    
    iex> String.slice("Hello world!", ..)
    "Hello world!"


### first..last
*(macro)* 


Creates a range from `first` to `last`.

If first is less than last, the range will be increasing from
first to last. If first is equal to last, the range will contain
one element, which is the number itself.

If first is more than last, the range will be decreasing from first
to last, albeit this behavior is deprecated. Instead prefer to
explicitly list the step with `first..last//-1`.

See the `Range` module for more information.

#### Examples

    iex> 0 in 1..3
    false
    iex> 2 in 1..3
    true
    
    iex> Enum.to_list(1..3)
    [1, 2, 3]


### first..last//step
*(since 1.12.0)* *(macro)* 


Creates a range from `first` to `last` with `step`.

See the `Range` module for more information.

#### Examples

    iex> 0 in 1..3//1
    false
    iex> 2 in 1..3//1
    true
    iex> 2 in 1..3//2
    false
    
    iex> Enum.to_list(1..3//1)
    [1, 2, 3]
    iex> Enum.to_list(1..3//2)
    [1, 3]
    iex> Enum.to_list(3..1//-1)
    [3, 2, 1]
    iex> Enum.to_list(1..0//1)
    []


### !value
*(macro)* 


Boolean "not" operator.

Receives any value (not just booleans) and returns `true` if `value`
is `false` or `nil`; returns `false` otherwise.

Not allowed in guard clauses.

#### Examples

    iex> !Enum.empty?([])
    false
    
    iex> !List.first([])
    true


### left &lt;&gt; right
*(macro)* 


Binary concatenation operator. Concatenates two binaries.

Raises an `ArgumentError` if one of the sides aren't binaries.

#### Examples

    iex> "foo" <> "bar"
    "foobar"

The `<>/2` operator can also be used in pattern matching (and guard clauses) as
long as the left argument is a literal binary:

    iex> "foo" <> x = "foobar"
    iex> x
    "bar"

`x <> "bar" = "foobar"` would result in an `ArgumentError` exception.


### left =~ right

```elixir
@spec String.t() =~ (String.t() | Regex.t()) :: boolean()
```

Text-based match operator. Matches the term on the `left`
against the regular expression or string on the `right`.

If `right` is a regular expression, returns `true` if `left` matches right.

If `right` is a string, returns `true` if `left` contains `right`.

#### Examples

    iex> "abcd" =~ ~r/c(d)/
    true
    
    iex> "abcd" =~ ~r/e/
    false
    
    iex> "abcd" =~ ~r//
    true
    
    iex> "abcd" =~ "bc"
    true
    
    iex> "abcd" =~ "ad"
    false
    
    iex> "abcd" =~ "abcd"
    true
    
    iex> "abcd" =~ ""
    true

For more information about regular expressions, please check the `Regex` module.


### @expr
*(macro)* 


Module attribute unary operator.

Reads and writes attributes in the current module.

The canonical example for attributes is annotating that a module
implements an OTP behaviour, such as `GenServer`:

    defmodule MyServer do
      @behaviour GenServer
      # ... callbacks ...
    end

By default Elixir supports all the module attributes supported by Erlang, but
custom attributes can be used as well:

    defmodule MyServer do
      @my_data 13
      IO.inspect(@my_data)
      #=> 13
    end

Unlike Erlang, such attributes are not stored in the module by default since
it is common in Elixir to use custom attributes to store temporary data that
will be available at compile-time. Custom attributes may be configured to
behave closer to Erlang by using `Module.register_attribute/3`.

> #### Prefixing module attributes {: .tip}
> 
> Libraries and frameworks should consider prefixing any
> module attributes that are private by underscore, such as `@_my_data`,
> so code completion tools do not show them on suggestions and prompts.

Finally, note that attributes can also be read inside functions:

    defmodule MyServer do
      @my_data 11
      def first_data, do: @my_data
      @my_data 13
      def second_data, do: @my_data
    end
    
    MyServer.first_data()
    #=> 11
    
    MyServer.second_data()
    #=> 13

It is important to note that reading an attribute takes a snapshot of
its current value. In other words, the value is read at compilation
time and not at runtime. Check the `Module` module for other functions
to manipulate module attributes.

#### Attention\! Multiple references of the same attribute

As mentioned above, every time you read a module attribute, a snapshot
of its current value is taken. Therefore, if you are storing large
values inside module attributes (for example, embedding external files
in module attributes), you should avoid referencing the same attribute
multiple times. For example, don't do this:

    @files %{
      example1: File.read!("lib/example1.data"),
      example2: File.read!("lib/example2.data")
    }
    
    def example1, do: @files[:example1]
    def example2, do: @files[:example2]

In the above, each reference to `@files` may end-up with a complete
and individual copy of the whole `@files` module attribute. Instead,
reference the module attribute once in a private function:

    @files %{
      example1: File.read!("lib/example1.data"),
      example2: File.read!("lib/example2.data")
    }
    
    defp files(), do: @files
    def example1, do: files()[:example1]
    def example2, do: files()[:example2]


### alias!(alias)
*(macro)* 


When used inside quoting, marks that the given alias should not
be hygienized. This means the alias will be expanded when
the macro is expanded.

Check `quote/2` for more information.


### apply(fun, args)

```elixir
@spec apply((... -&gt; any()), [any()]) :: any()
```

Invokes the given anonymous function `fun` with the list of
arguments `args`.

If the number of arguments is known at compile time, prefer
`fun.(arg_1, arg_2, ..., arg_n)` as it is clearer than
`apply(fun, [arg_1, arg_2, ..., arg_n])`.

Inlined by the compiler.

#### Examples

    iex> apply(fn x -> x * 2 end, [2])
    4


### apply(module, function_name, args)

```elixir
@spec apply(module(), function_name :: atom(), [any()]) :: any()
```

Invokes the given function from `module` with the list of
arguments `args`.

`apply/3` is used to invoke functions where the module, function
name or arguments are defined dynamically at runtime. For this
reason, you can't invoke macros using `apply/3`, only functions.

If the number of arguments and the function name are known at compile time,
prefer `module.function(arg_1, arg_2, ..., arg_n)` as it is clearer than
`apply(module, :function, [arg_1, arg_2, ..., arg_n])`.

`apply/3` cannot be used to call private functions.

Inlined by the compiler.

#### Examples

    iex> apply(Enum, :reverse, [[1, 2, 3]])
    [3, 2, 1]


### binary_slice(binary, range)
*(since 1.14.0)* 


Returns a binary from the offset given by the start of the
range to the offset given by the end of the range.

If the start or end of the range are negative, they are converted
into positive indices based on the binary size. For example,
`-1` means the last byte of the binary.

This is similar to `binary_part/3` except that it works with ranges
and it is not allowed in guards.

This function works with bytes. For a slicing operation that
considers characters, see `String.slice/2`.

#### Examples

    iex> binary_slice("elixir", 0..5)
    "elixir"
    iex> binary_slice("elixir", 1..3)
    "lix"
    iex> binary_slice("elixir", 1..10)
    "lixir"
    
    iex> binary_slice("elixir", -4..-1)
    "ixir"
    iex> binary_slice("elixir", -4..6)
    "ixir"
    iex> binary_slice("elixir", -10..10)
    "elixir"

For ranges where `start > stop`, you need to explicitly
mark them as increasing:

    iex> binary_slice("elixir", 2..-1//1)
    "ixir"
    iex> binary_slice("elixir", 1..-2//1)
    "lixi"

You can use `../0` as a shortcut for `0..-1//1`, which returns
the whole binary as is:

    iex> binary_slice("elixir", ..)
    "elixir"

The step can be any positive number. For example, to
get every 2 characters of the binary:

    iex> binary_slice("elixir", 0..-1//2)
    "eii"

If the first position is after the string ends or after
the last position of the range, it returns an empty string:

    iex> binary_slice("elixir", 10..3//1)
    ""
    iex> binary_slice("elixir", -10..-7)
    ""
    iex> binary_slice("a", 1..1500)
    ""


### binary_slice(binary, start, size)
*(since 1.14.0)* 


Returns a binary starting at the offset `start` and of the given `size`.

This is similar to `binary_part/3` except that if `start + size`
is greater than the binary size, it automatically clips it to
the binary size instead of raising. Opposite to `binary_part/3`,
this function is not allowed in guards.

This function works with bytes. For a slicing operation that
considers characters, see `String.slice/3`.

#### Examples

    iex> binary_slice("elixir", 0, 6)
    "elixir"
    iex> binary_slice("elixir", 0, 5)
    "elixi"
    iex> binary_slice("elixir", 1, 4)
    "lixi"
    iex> binary_slice("elixir", 0, 10)
    "elixir"

If `start` is negative, it is normalized against the binary
size and clamped to 0:

    iex> binary_slice("elixir", -3, 10)
    "xir"
    iex> binary_slice("elixir", -10, 10)
    "elixir"

If the `size` is zero, an empty binary is returned:

    iex> binary_slice("elixir", 1, 0)
    ""

If `start` is greater than or equal to the binary size,
an empty binary is returned:

    iex> binary_slice("elixir", 10, 10)
    ""


### binding(context \\ nil)
*(macro)* 


Returns the binding for the given context as a keyword list.

In the returned result, keys are variable names and values are the
corresponding variable values.

If the given `context` is `nil` (by default it is), the binding for the
current context is returned.

#### Examples

    iex> x = 1
    iex> binding()
    [x: 1]
    iex> x = 2
    iex> binding()
    [x: 2]
    
    iex> binding(:foo)
    []
    iex> var!(x, :foo) = 1
    1
    iex> binding(:foo)
    [x: 1]


### dbg(code \\ quote do
  binding()
end, options \\ [])
*(since 1.14.0)* *(macro)* 


Debugs the given `code`.

`dbg/2` can be used to debug the given `code` through a configurable debug function.
It returns the result of the given code.

#### Examples

Let's take this call to `dbg/2`:

    dbg(Atom.to_string(:debugging))
    #=> "debugging"

It returns the string `"debugging"`, which is the result of the `Atom.to_string/1` call.
Additionally, the call above prints:

    [my_file.ex:10: MyMod.my_fun/0]
    Atom.to_string(:debugging) #=> "debugging"

The default debugging function prints additional debugging info when dealing with
pipelines. It prints the values at every "step" of the pipeline.

    "Elixir is cool!"
    |> String.trim_trailing("!")
    |> String.split()
    |> List.first()
    |> dbg()
    #=> "Elixir"

The code above prints:

    [my_file.ex:10: MyMod.my_fun/0]
    "Elixir is cool!" #=> "Elixir is cool!"
    |> String.trim_trailing("!") #=> "Elixir is cool"
    |> String.split() #=> ["Elixir", "is", "cool"]
    |> List.first() #=> "Elixir"

With no arguments, `dbg()` debugs information about the current binding. See `binding/1`.

#### `dbg` inside IEx

You can enable IEx to replace `dbg` with its `IEx.pry/0` backend by calling:

    $ iex --dbg pry

In such cases, `dbg` will start a `pry` session where you can interact with
the imports, aliases, and variables of the current environment at the location
of the `dbg` call.

If you call `dbg` at the end of a pipeline (using `|>`) within IEx, you are able
to go through each step of the pipeline one by one by entering "next" (or "n").

Note `dbg` only supports stepping for pipelines (in other words, it can only
step through the code it sees). For general stepping, you can set breakpoints
using `IEx.break!/4`.

For more information, [see IEx documentation](https://hexdocs.pm/iex/IEx.html#module-dbg-and-breakpoints).

#### Configuring the debug function

One of the benefits of `dbg/2` is that its debugging logic is configurable,
allowing tools to extend `dbg` with enhanced behaviour. This is done, for
example, by `IEx` which extends `dbg` with an interactive shell where you
can directly inspect and access values.

The debug function can be configured at compile time through the `:dbg_callback`
key of the `:elixir` application. The debug function must be a
`{module, function, args}` tuple. The `function` function in `module` will be
invoked with three arguments *prepended* to `args`:

1.  The AST of `code`
2.  The AST of `options`
3.  The `Macro.Env` environment of where `dbg/2` is invoked

The debug function is invoked at compile time and it must also return an AST.
The AST is expected to ultimately return the result of evaluating the debugged
expression.

Here's a simple example:

    defmodule MyMod do
      def debug_fun(code, options, caller, device) do
        quote do
          result = unquote(code)
          IO.inspect(unquote(device), result, label: unquote(Macro.to_string(code)))
        end
      end
    end

To configure the debug function:

    # In config/config.exs
    config :elixir, :dbg_callback, {MyMod, :debug_fun, [:stdio]}

##### Default debug function

By default, the debug function we use is `Macro.dbg/3`. It just prints
information about the code to standard output and returns the value
returned by evaluating `code`. `options` are used to control how terms
are inspected. They are the same options accepted by `inspect/2`.


### def(call, expr \\ nil)
*(macro)* 


Defines a public function with the given name and body.

#### Examples

    defmodule Foo do
      def bar, do: :baz
    end
    
    Foo.bar()
    #=> :baz

A function that expects arguments can be defined as follows:

    defmodule Foo do
      def sum(a, b) do
        a + b
      end
    end

In the example above, a `sum/2` function is defined; this function receives
two arguments and returns their sum.

#### Default arguments

`\\` is used to specify a default value for a parameter of a function. For
example:

    defmodule MyMath do
      def multiply_by(number, factor \\ 2) do
        number * factor
      end
    end
    
    MyMath.multiply_by(4, 3)
    #=> 12
    
    MyMath.multiply_by(4)
    #=> 8

The compiler translates this into multiple functions with different arities,
here `MyMath.multiply_by/1` and `MyMath.multiply_by/2`, that represent cases when
arguments for parameters with default values are passed or not passed.

When defining a function with default arguments as well as multiple
explicitly declared clauses, you must write a function head that declares the
defaults. For example:

    defmodule MyString do
      def join(string1, string2 \\ nil, separator \\ " ")
    
      def join(string1, nil, _separator) do
        string1
      end
    
      def join(string1, string2, separator) do
        string1 <> separator <> string2
      end
    end

Note that `\\` can't be used with anonymous functions because they
can only have a sole arity.

##### Keyword lists with default arguments

Functions containing many arguments can benefit from using `Keyword`
lists to group and pass attributes as a single value.

    defmodule MyConfiguration do
      @default_opts [storage: "local"]
    
      def configure(resource, opts \\ []) do
        opts = Keyword.merge(@default_opts, opts)
        storage = opts[:storage]
        # ...
      end
    end

The difference between using `Map` and `Keyword` to store many
arguments is `Keyword`'s keys:

- must be atoms
- can be given more than once
- ordered, as specified by the developer

#### Function names

Function and variable names in Elixir must start with an underscore or a
Unicode letter that is not in uppercase or titlecase. They may continue
using a sequence of Unicode letters, numbers, and underscores. They may
end in `?` or `!`. Elixir's [Naming Conventions](naming-conventions.md)
suggest for function and variable names to be written in the `snake_case`
format.

#### `rescue`/`catch`/`after`/`else`

Function bodies support `rescue`, `catch`, `after`, and `else` as `try/1`
does (known as "implicit try"). For example, the following two functions are equivalent:

    def convert(number) do
      try do
        String.to_integer(number)
      rescue
        e in ArgumentError -> {:error, e.message}
      end
    end
    
    def convert(number) do
      String.to_integer(number)
    rescue
      e in ArgumentError -> {:error, e.message}
    end


### defdelegate(funs, opts)
*(macro)* 


Defines a function that delegates to another module.

Functions defined with `defdelegate/2` are public and can be invoked from
outside the module they're defined in, as if they were defined using `def/2`.
Therefore, `defdelegate/2` is about extending the current module's public API.
If what you want is to invoke a function defined in another module without
using its full module name, then use `alias/2` to shorten the module name or use
`import/2` to be able to invoke the function without the module name altogether.

Delegation only works with functions; delegating macros is not supported.

Check `def/2` for rules on naming and default arguments.

#### Options

- `:to` - the module to dispatch to.

- `:as` - the function to call on the target given in `:to`.
  This parameter is optional and defaults to the name being
  delegated (`funs`).

#### Examples

    defmodule MyList do
      defdelegate reverse(list), to: Enum
      defdelegate other_reverse(list), to: Enum, as: :reverse
    end
    
    MyList.reverse([1, 2, 3])
    #=> [3, 2, 1]
    
    MyList.other_reverse([1, 2, 3])
    #=> [3, 2, 1]


### defexception(fields)
*(macro)* 


Defines an exception.

Exceptions are structs backed by a module that implements
the `Exception` behaviour. The `Exception` behaviour requires
two functions to be implemented:

- [`exception/1`](\`c:Exception.exception/1\`) - receives the arguments given to `raise/2`
  and returns the exception struct. The default implementation
  accepts either a set of keyword arguments that is merged into
  the struct or a string to be used as the exception's message.

- [`message/1`](\`c:Exception.message/1\`) - receives the exception struct and must return its
  message. Most commonly exceptions have a message field which
  by default is accessed by this function. However, if an exception
  does not have a message field, this function must be explicitly
  implemented.

Since exceptions are structs, the API supported by `defstruct/1`
is also available in `defexception/1`.

#### Raising exceptions

The most common way to raise an exception is via `raise/2`:

    defmodule MyAppError do
      defexception [:message]
    end
    
    value = [:hello]
    
    raise MyAppError,
      message: "did not get what was expected, got: #{inspect(value)}"

In many cases it is more convenient to pass the expected value to
`raise/2` and generate the message in the `c:Exception.exception/1` callback:

    defmodule MyAppError do
      defexception [:message]
    
      @impl true
      def exception(value) do
        msg = "did not get what was expected, got: #{inspect(value)}"
        %MyAppError{message: msg}
      end
    end
    
    raise MyAppError, value

The example above shows the preferred strategy for customizing
exception messages.


### defguard(guard)
*(since 1.6.0)* *(macro)* 
```elixir
@spec defguard(Macro.t()) :: Macro.t()
```

Defines a macro suitable for use in guard expressions.

It raises at compile time if the `guard` uses expressions that aren't
allowed in [guard clauses](patterns-and-guards.html#guards),
and otherwise creates a macro that can be used both inside or outside guards.

When defining your own guards, consider the
[naming conventions](naming-conventions.html#is_-prefix-is_foo)
around boolean-returning guards.

#### Example

    defmodule Integer.Guards do
      defguard is_even(value) when is_integer(value) and rem(value, 2) == 0
    end
    
    defmodule Collatz do
      @moduledoc "Tools for working with the Collatz sequence."
      import Integer.Guards
    
      @doc "Determines the number of steps `n` takes to reach `1`."
      # If this function never converges, please let me know what `n` you used.
      def converge(n) when n > 0, do: step(n, 0)
    
      defp step(1, step_count) do
        step_count
      end
    
      defp step(n, step_count) when is_even(n) do
        step(div(n, 2), step_count + 1)
      end
    
      defp step(n, step_count) do
        step(3 * n + 1, step_count + 1)
      end
    end


### defguardp(guard)
*(since 1.6.0)* *(macro)* 
```elixir
@spec defguardp(Macro.t()) :: Macro.t()
```

Defines a private macro suitable for use in guard expressions.

It raises at compile time if the `guard` uses expressions that aren't
allowed in [guard clauses](patterns-and-guards.html#guards),
and otherwise creates a private macro that can be used
both inside or outside guards in the current module.

When defining your own guards, consider the
[naming conventions](naming-conventions.html#is_-prefix-is_foo)
around boolean-returning guards.

Similar to `defmacrop/2`, `defguardp/1` must be defined before its use
in the current module.


### defimpl(name, opts, do_block \\ [])
*(macro)* 


Defines an implementation for the given protocol.

See the `Protocol` module for more information.


### defmacro(call, expr \\ nil)
*(macro)* 


Defines a public macro with the given name and body.

Macros must be defined before its usage.

Check `def/2` for rules on naming and default arguments.

#### Examples

    defmodule MyLogic do
      defmacro unless(expr, opts) do
        quote do
          if !unquote(expr), unquote(opts)
        end
      end
    end
    
    require MyLogic
    
    MyLogic.unless false do
      IO.puts("It works")
    end


### defmacrop(call, expr \\ nil)
*(macro)* 


Defines a private macro with the given name and body.

Private macros are only accessible from the same module in which they are
defined.

Private macros must be defined before its usage.

Check `defmacro/2` for more information, and check `def/2` for rules on
naming and default arguments.


### defmodule(alias, do_block)
*(macro)* 


Defines a module given by name with the given contents.

This macro defines a module with the given `alias` as its name and with the
given contents. It returns a tuple with four elements:

- `:module`
- the module name
- the binary contents of the module
- the result of evaluating the contents block

#### Examples

    defmodule Number do
      def one, do: 1
      def two, do: 2
    end
    #=> {:module, Number, <<70, 79, 82, ...>>, {:two, 0}}
    
    Number.one()
    #=> 1
    
    Number.two()
    #=> 2

#### Module names and aliases

Module names (and aliases) must start with an ASCII uppercase character which
may be followed by any ASCII letter, number, or underscore. Elixir's
[Naming Conventions](naming-conventions.md) suggest for module names and aliases
to be written in the `CamelCase` format.

You can also use atoms as the module name, although they must only contain ASCII
characters.

#### Nesting

Nesting a module inside another module affects the name of the nested module:

    defmodule Foo do
      defmodule Bar do
      end
    end

In the example above, two modules - `Foo` and `Foo.Bar` - are created.
When nesting, Elixir automatically creates an alias to the inner module,
allowing the second module `Foo.Bar` to be accessed as `Bar` in the same
lexical scope where it's defined (the `Foo` module). This only happens
if the nested module is defined via an alias.

If the `Foo.Bar` module is moved somewhere else, the references to `Bar` in
the `Foo` module need to be updated to the fully-qualified name (`Foo.Bar`) or
an alias has to be explicitly set in the `Foo` module with the help of
`alias/2`.

    defmodule Foo.Bar do
      # code
    end
    
    defmodule Foo do
      alias Foo.Bar
      # code here can refer to "Foo.Bar" as just "Bar"
    end

#### Dynamic names

Elixir module names can be dynamically generated. This is very
useful when working with macros. For instance, one could write:

    defmodule Module.concat(["Foo", "Bar"]) do
      # contents ...
    end

Elixir will accept any module name as long as the expression passed as the
first argument to `defmodule/2` evaluates to an atom.
Note that, when a dynamic name is used, Elixir won't nest the name under
the current module nor automatically set up an alias.

#### Reserved module names

If you attempt to define a module that already exists, you will get a
warning saying that a module has been redefined.

There are some modules that Elixir does not currently implement but it
may be implement in the future. Those modules are reserved and defining
them will result in a compilation error:

    defmodule Any do
      # code
    end
    ** (CompileError) iex:1: module Any is reserved and cannot be defined

Elixir reserves the following module names: `Elixir`, `Any`, `BitString`,
`PID`, and `Reference`.


### defoverridable(keywords_or_behaviour)
*(macro)* 


Makes the given definitions in the current module overridable.

If the user defines a new function or macro with the same name
and arity, then the overridable ones are discarded. Otherwise, the
original definitions are used.

It is possible for the overridden definition to have a different visibility
than the original: a public function can be overridden by a private
function and vice-versa.

Macros cannot be overridden as functions and vice-versa.

#### Example

    defmodule DefaultMod do
      defmacro __using__(_opts) do
        quote do
          def test(x, y) do
            x + y
          end
    
          defoverridable test: 2
        end
      end
    end
    
    defmodule ChildMod do
      use DefaultMod
    
      def test(x, y) do
        x * y + super(x, y)
      end
    end

As seen as in the example above, `super` can be used to call the default
implementation.

> #### Disclaimer {: .tip}
> 
> Use `defoverridable` with care. If you need to define multiple modules
> with the same behaviour, it may be best to move the default implementation
> to the caller, and check if a callback exists via `Code.ensure_loaded?/1` and
> `function_exported?/3`.
> 
> For example, in the example above, imagine there is a module that calls the
> `test/2` function. This module could be defined as such:
> 
>     defmodule CallsTest do
>       def receives_module_and_calls_test(module, x, y) do
>         if Code.ensure_loaded?(module) and function_exported?(module, :test, 2) do
>           module.test(x, y)
>         else
>           x + y
>         end
>       end
>     end

#### Example with behaviour

You can also pass a behaviour to `defoverridable` and it will mark all of the
callbacks in the behaviour as overridable:

    defmodule Behaviour do
      @callback test(number(), number()) :: number()
    end
    
    defmodule DefaultMod do
      defmacro __using__(_opts) do
        quote do
          @behaviour Behaviour
    
          def test(x, y) do
            x + y
          end
    
          defoverridable Behaviour
        end
      end
    end
    
    defmodule ChildMod do
      use DefaultMod
    
      def test(x, y) do
        x * y + super(x, y)
      end
    end


### defp(call, expr \\ nil)
*(macro)* 


Defines a private function with the given name and body.

Private functions are only accessible from within the module in which they are
defined. Trying to access a private function from outside the module it's
defined in results in an `UndefinedFunctionError` exception.

Check `def/2` for more information.

#### Examples

    defmodule Foo do
      def bar do
        sum(1, 2)
      end
    
      defp sum(a, b), do: a + b
    end
    
    Foo.bar()
    #=> 3
    
    Foo.sum(1, 2)
    ** (UndefinedFunctionError) undefined function Foo.sum/2


### defprotocol(name, do_block)
*(macro)* 


Defines a protocol.

See the `Protocol` module for more information.


### defstruct(fields)
*(macro)* 


Defines a struct.

A struct is a tagged map that allows developers to provide
default values for keys, tags to be used in polymorphic
dispatches and compile time assertions.

It is only possible to define a struct per module, as the
struct is tied to the module itself.

#### Examples

    defmodule User do
      defstruct name: nil, age: nil
    end

Struct fields are evaluated at compile-time, which allows
them to be dynamic. In the example below, `10 + 11` is
evaluated at compile-time and the age field is stored
with value `21`:

    defmodule User do
      defstruct name: nil, age: 10 + 11
    end

The `fields` argument is usually a keyword list with field names
as atom keys and default values as corresponding values. `defstruct/1`
also supports a list of atoms as its argument: in that case, the atoms
in the list will be used as the struct's field names and they will all
default to `nil`.

    defmodule Post do
      defstruct [:title, :content, :author]
    end

Add documentation to a struct with the `@doc` attribute, like a function.

    defmodule Post do
      @doc "A post. The content should be valid Markdown."
      defstruct [:title, :content, :author]
    end

Once a struct is defined, it is possible to create them as follows:

    %Post{title: "Hello world!"}

For more information on creating, updating, and pattern matching on
structs, please check `%/2`.

#### Deriving

Although structs are maps, by default structs do not implement
any of the protocols implemented for maps. For example, attempting
to use a protocol with the `User` struct leads to an error:

    john = %User{name: "John"}
    MyProtocol.call(john)
    ** (Protocol.UndefinedError) protocol MyProtocol not implemented for %User{...}

`defstruct/1`, however, allows protocol implementations to be
*derived*. This can be done by defining a `@derive` attribute as a
list before invoking `defstruct/1`:

    defmodule User do
      @derive MyProtocol
      defstruct name: nil, age: nil
    end
    
    MyProtocol.call(john) # it works!

A common example is to `@derive` the `Inspect` protocol to hide certain fields
when the struct is printed:

    defmodule User do
      @derive {Inspect, only: :name}
      defstruct name: nil, age: nil
    end

For each protocol in `@derive`, Elixir will verify if the protocol
has implemented the `c:Protocol.__deriving__/2` callback. If so,
the callback will be invoked and it should define the implementation
module. Otherwise an implementation that simply points to the `Any`
implementation is automatically derived. For more information, see
`Protocol.derive/3`.

#### Enforcing keys

When building a struct, Elixir will automatically guarantee all keys
belong to the struct:

    %User{name: "john", unknown: :key}
    ** (KeyError) key :unknown not found in: %User{age: 21, name: nil}

Elixir also allows developers to enforce that certain keys must always be
given when building the struct:

    defmodule User do
      @enforce_keys [:name]
      defstruct name: nil, age: 10 + 11
    end

Now trying to build a struct without the name key will fail:

    %User{age: 21}
    ** (ArgumentError) the following keys must also be given when building struct User: [:name]

Keep in mind `@enforce_keys` is a simple compile-time guarantee
to aid developers when building structs. It is not enforced on
updates and it does not provide any sort of value-validation.

#### Types

It is recommended to define types for structs. By convention, such a type
is called `t`. To define a struct inside a type, the struct literal syntax
is used:

    defmodule User do
      defstruct name: "John", age: 25
      @type t :: %__MODULE__{name: String.t(), age: non_neg_integer}
    end

It is recommended to only use the struct syntax when defining the struct's
type. When referring to another struct, it's better to use `User.t()` instead of
`%User{}`.

The types of the struct fields that are not included in `%User{}` default to
`term()` (see `t:term/0`).

Structs whose internal structure is private to the local module (pattern
matching them or directly accessing their fields should not be allowed) should
use the `@opaque` attribute. Structs whose internal structure is public should
use `@type`.


### destructure(left, right)
*(macro)* 


Destructures two lists, assigning each term in the
right one to the matching term in the left one.

Unlike pattern matching via `=`, if the sizes of the left
and right lists don't match, destructuring simply stops
instead of raising an error.

#### Examples

    iex> destructure([x, y, z], [1, 2, 3, 4, 5])
    iex> {x, y, z}
    {1, 2, 3}

In the example above, even though the right list has more entries than the
left one, destructuring works fine. If the right list is smaller, the
remaining elements are simply set to `nil`:

    iex> destructure([x, y, z], [1])
    iex> {x, y, z}
    {1, nil, nil}

The left-hand side supports any expression you would use
on the left-hand side of a match:

    x = 1
    destructure([^x, y, z], [1, 2, 3])

The example above will only work if `x` matches the first value in the right
list. Otherwise, it will raise a `MatchError` (like the `=` operator would
do).


### exit(reason)

```elixir
@spec exit(term()) :: no_return()
```

Stops the execution of the calling process with the given reason.

Since evaluating this function causes the process to terminate,
it has no return value.

Inlined by the compiler.

#### Examples

When a process reaches its end, by default it exits with
reason `:normal`. You can also call `exit/1` explicitly if you
want to terminate a process but not signal any failure:

    exit(:normal)

In case something goes wrong, you can also use `exit/1` with
a different reason:

    exit(:seems_bad)

If the exit reason is not `:normal`, all the processes linked to the process
that exited will crash (unless they are trapping exits).

#### OTP exits

Exits are used by the OTP to determine if a process exited abnormally
or not. The following exits are considered "normal":

- `exit(:normal)`
- `exit(:shutdown)`
- `exit({:shutdown, term})`

Exiting with any other reason is considered abnormal and treated
as a crash. This means the default supervisor behavior kicks in,
error reports are emitted, and so forth.

This behavior is relied on in many different places. For example,
`ExUnit` uses `exit(:shutdown)` when exiting the test process to
signal linked processes, supervision trees and so on to politely
shut down too.

#### CLI exits

Building on top of the exit signals mentioned above, if the
process started by the command line exits with any of the three
reasons above, its exit is considered normal and the Operating
System process will exit with status 0.

It is, however, possible to customize the operating system exit
signal by invoking:

    exit({:shutdown, integer})

This will cause the operating system process to exit with the status given by
`integer` while signaling all linked Erlang processes to politely
shut down.

Any other exit reason will cause the operating system process to exit with
status `1` and linked Erlang processes to crash.


### function_exported?(module, function, arity)

```elixir
@spec function_exported?(module(), atom(), arity()) :: boolean()
```

Returns `true` if `module` is loaded and contains a
public `function` with the given `arity`, otherwise `false`.

Note that this function does not load the module in case
it is not loaded. Check `Code.ensure_loaded/1` for more
information.

Inlined by the compiler.

#### Examples

    iex> function_exported?(Enum, :map, 2)
    true
    
    iex> function_exported?(Enum, :map, 10)
    false
    
    iex> function_exported?(List, :to_string, 1)
    true


### get_and_update_in(path, fun)
*(macro)* 


Gets a value and updates a nested data structure via the given `path`.

This is similar to `get_and_update_in/3`, except the path is extracted
via a macro rather than passing a list. For example:

    get_and_update_in(opts[:foo][:bar], &{&1, &1 + 1})

Is equivalent to:

    get_and_update_in(opts, [:foo, :bar], &{&1, &1 + 1})

This also works with nested structs and the `struct.path.to.value` way to specify
paths:

    get_and_update_in(struct.foo.bar, &{&1, &1 + 1})

Note that in order for this macro to work, the complete path must always
be visible by this macro. See the "Paths" section below.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> get_and_update_in(users["john"].age, &{&1, &1 + 1})
    {27, %{"john" => %{age: 28}, "meg" => %{age: 23}}}

#### Paths

A path may start with a variable, local or remote call, and must be
followed by one or more:

- `foo[bar]` - accesses the key `bar` in `foo`; in case `foo` is nil,
  `nil` is returned

- `foo.bar` - accesses a map/struct field; in case the field is not
  present, an error is raised

Here are some valid paths:

    users["john"][:age]
    users["john"].age
    User.all()["john"].age
    all_users()["john"].age

Here are some invalid ones:

    # Does a remote call after the initial value
    users["john"].do_something(arg1, arg2)
    
    # Does not access any key or field
    users


### get_and_update_in(data, keys, fun)

```elixir
@spec get_and_update_in(
  structure,
  keys,
  (term() | nil -&gt; {current_value, new_value} | :pop)
) :: {current_value, new_structure :: structure}
when structure: Access.t(),
     keys: [term(), ...],
     current_value: Access.value(),
     new_value: Access.value()
```

Gets a value and updates a nested structure.

`data` is a nested structure (that is, a map, keyword
list, or struct that implements the `Access` behaviour).

The `fun` argument receives the value of `key` (or `nil` if `key`
is not present) and must return one of the following values:

- a two-element tuple `{current_value, new_value}`. In this case,
  `current_value` is the retrieved value which can possibly be operated on before
  being returned. `new_value` is the new value to be stored under `key`.

- `:pop`, which implies that the current value under `key`
  should be removed from the structure and returned.

This function uses the `Access` module to traverse the structures
according to the given `keys`, unless the `key` is a function,
which is detailed in a later section.

#### Examples

This function is useful when there is a need to retrieve the current
value (or something calculated in function of the current value) and
update it at the same time. For example, it could be used to read the
current age of a user while increasing it by one in one pass:

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> get_and_update_in(users, ["john", :age], &{&1, &1 + 1})
    {27, %{"john" => %{age: 28}, "meg" => %{age: 23}}}

Note the current value given to the anonymous function may be `nil`.
If any of the intermediate values are nil, it will raise:

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> get_and_update_in(users, ["jane", :age], &{&1, &1 + 1})
    ** (ArgumentError) could not put/update key :age on a nil value

#### Functions as keys

If a key is a function, the function will be invoked passing three
arguments:

- the operation (`:get_and_update`)
- the data to be accessed
- a function to be invoked next

This means `get_and_update_in/3` can be extended to provide custom
lookups. The downside is that functions cannot be stored as keys
in the accessed data structures.

When one of the keys is a function, the function is invoked.
In the example below, we use a function to get and increment all
ages inside a list:

    iex> users = [%{name: "john", age: 27}, %{name: "meg", age: 23}]
    iex> all = fn :get_and_update, data, next ->
    ...>   data |> Enum.map(next) |> Enum.unzip()
    ...> end
    iex> get_and_update_in(users, [all, :age], &{&1, &1 + 1})
    {[27, 23], [%{name: "john", age: 28}, %{name: "meg", age: 24}]}

If the previous value before invoking the function is `nil`,
the function *will* receive `nil` as a value and must handle it
accordingly (be it by failing or providing a sane default).

The `Access` module ships with many convenience accessor functions,
like the `all` anonymous function defined above. See `Access.all/0`,
`Access.key/2`, and others as examples.


### get_in(path)
*(macro)* 


Gets a key from the nested structure via the given `path`, with
nil-safe handling.

This is similar to `get_in/2`, except the path is extracted via
a macro rather than passing a list. For example:

    get_in(opts[:foo][:bar])

Is equivalent to:

    get_in(opts, [:foo, :bar])

Additionally, this macro can traverse structs:

    get_in(struct.foo.bar)

In case any of the keys returns `nil`, then `nil` will be returned
and `get_in/1` won't traverse any further.

Note that in order for this macro to work, the complete path must always
be visible by this macro. For more information about the supported path
expressions, please check `get_and_update_in/2` docs.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> get_in(users["john"].age)
    27
    iex> get_in(users["unknown"].age)
    nil


### get_in(data, keys)

```elixir
@spec get_in(Access.t(), [term(), ...]) :: term()
```

Gets a value from a nested structure with nil-safe handling.

Uses the `Access` module to traverse the structures
according to the given `keys`, unless the `key` is a
function, which is detailed in a later section.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> get_in(users, ["john", :age])
    27
    iex> # Equivalent to:
    iex> users["john"][:age]
    27

`get_in/2` can also use the accessors in the `Access` module
to traverse more complex data structures. For example, here we
use `Access.all/0` to traverse a list:

    iex> users = [%{name: "john", age: 27}, %{name: "meg", age: 23}]
    iex> get_in(users, [Access.all(), :age])
    [27, 23]

In case any of the components returns `nil`, `nil` will be returned
and `get_in/2` won't traverse any further:

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> get_in(users, ["unknown", :age])
    nil
    iex> # Equivalent to:
    iex> users["unknown"][:age]
    nil

#### Functions as keys

If a key given to `get_in/2` is a function, the function will be invoked
passing three arguments:

- the operation (`:get`)
- the data to be accessed
- a function to be invoked next

This means `get_in/2` can be extended to provide custom lookups.
That's precisely how the `Access.all/0` key in the previous section
behaves. For example, we can manually implement such traversal as
follows:

    iex> users = [%{name: "john", age: 27}, %{name: "meg", age: 23}]
    iex> all = fn :get, data, next -> Enum.map(data, next) end
    iex> get_in(users, [all, :age])
    [27, 23]

The `Access` module ships with many convenience accessor functions.
See `Access.all/0`, `Access.key/2`, and others as examples.

#### Working with structs

By default, structs do not implement the `Access` behaviour required
by this function. Therefore, you can't do this:

    get_in(some_struct, [:some_key, :nested_key])

There are two alternatives. Given structs have predefined keys,
we can use the `struct.field` notation:

    some_struct.some_key.nested_key

However, the code above will fail if any of the values return `nil`.
If you also want to handle nil values, you can use `get_in/1`:

    get_in(some_struct.some_key.nested_key)

Pattern-matching is another option for handling such cases,
which can be especially useful if you want to match on several
fields at once or provide custom return values:

    case some_struct do
      %{some_key: %{nested_key: value}} -> value
      %{} -> nil
    end


### if(condition, clauses)
*(macro)* 


Provides an `if/2` macro.

This macro expects the first argument to be a condition and the second
argument to be a keyword list. Generally speaking, Elixir developers
prefer to use pattern matching and guards in function definitions and
`case/2`, as they are succinct and precise. However, not all conditions
can be expressed through patterns and guards, which makes `if/2` a viable
alternative.

Similar to `case/2`, any assignment in the condition will be available
on both clauses, as well as after the `if` expression.

#### One-liner examples

    if(foo, do: bar)

In the example above, `bar` will be returned if `foo` evaluates to
a truthy value (neither `false` nor `nil`). Otherwise, `nil` will be
returned.

An `else` option can be given to specify the opposite:

    if(foo, do: bar, else: baz)

#### Blocks examples

It's also possible to pass a block to the `if/2` macro. The first
example above would be translated to:

    if foo do
      bar
    end

Note that `do`-`end` become delimiters. The second example would
translate to:

    if foo do
      bar
    else
      baz
    end

If you find yourself nesting conditionals inside conditionals,
consider using `cond/1`.


### inspect(term, opts \\ [])

```elixir
@spec inspect(
  Inspect.t(),
  keyword()
) :: String.t()
```

Inspects the given argument according to the `Inspect` protocol.
The second argument is a keyword list with options to control
inspection.

#### Options

`inspect/2` accepts a list of options that are internally
translated to an `Inspect.Opts` struct. Check the docs for
`Inspect.Opts` to see the supported options.

#### Examples

    iex> inspect(:foo)
    ":foo"
    
    iex> inspect([1, 2, 3, 4, 5], limit: 3)
    "[1, 2, 3, ...]"
    
    iex> inspect([1, 2, 3], pretty: true, width: 0)
    "[1,\n 2,\n 3]"
    
    iex> inspect("olá" <> <<0>>)
    "<<111, 108, 195, 161, 0>>"
    
    iex> inspect("olá" <> <<0>>, binaries: :as_strings)
    "\"olá\\0\""
    
    iex> inspect("olá", binaries: :as_binaries)
    "<<111, 108, 195, 161>>"
    
    iex> inspect(~c"bar")
    "~c\"bar\""
    
    iex> inspect([0 | ~c"bar"])
    "[0, 98, 97, 114]"
    
    iex> inspect(100, base: :octal)
    "0o144"
    
    iex> inspect(100, base: :hex)
    "0x64"

Note that the `Inspect` protocol does not necessarily return a valid
representation of an Elixir term. In such cases, the inspected result
must start with `#`. For example, inspecting a function will return:

    inspect(fn a, b -> a + b end)
    #=> #Function<...>

The `Inspect` protocol can be derived to hide certain fields
from structs, so they don't show up in logs, inspects and similar.
See the "Deriving" section of the documentation of the `Inspect`
protocol for more information.


### macro_exported?(module, macro, arity)

```elixir
@spec macro_exported?(module(), atom(), arity()) :: boolean()
```

Returns `true` if `module` is loaded and contains a
public `macro` with the given `arity`, otherwise `false`.

Note that this function does not load the module in case
it is not loaded. Check `Code.ensure_loaded/1` for more
information.

If `module` is an Erlang module (as opposed to an Elixir module), this
function always returns `false`.

#### Examples

    iex> macro_exported?(Kernel, :use, 2)
    true
    
    iex> macro_exported?(:erlang, :abs, 1)
    false


### make_ref()

```elixir
@spec make_ref() :: reference()
```

Returns an almost unique reference.

The returned reference will re-occur after approximately 2^82 calls;
therefore it is unique enough for practical purposes.

Inlined by the compiler.

#### Examples

    make_ref()
    #=> #Reference<0.0.0.135>


### match?(pattern, expr)
*(macro)* 


A convenience macro that checks if the right side (an expression) matches the
left side (a pattern).

#### Examples

    iex> match?(1, 1)
    true
    
    iex> match?({1, _}, {1, 2})
    true
    
    iex> map = %{a: 1, b: 2}
    iex> match?(%{a: _}, map)
    true
    
    iex> a = 1
    iex> match?(^a, 1)
    true

`match?/2` is very useful when filtering or finding a value in an enumerable:

    iex> list = [a: 1, b: 2, a: 3]
    iex> Enum.filter(list, &match?({:a, _}, &1))
    [a: 1, a: 3]

Guard clauses can also be given to the match:

    iex> list = [a: 1, b: 2, a: 3]
    iex> Enum.filter(list, &match?({:a, x} when x < 2, &1))
    [a: 1]

Variables assigned in the match will not be available outside of the
function call (unlike regular pattern matching with the `=` operator):

    iex> match?(_x, 1)
    true
    iex> binding()
    []

#### Values vs patterns

Remember the pin operator matches *values*, not *patterns*.
Passing a variable as the pattern will always return `true` and will
result in a warning that the variable is unused:

    # don't do this
    pattern = %{a: :a}
    match?(pattern, %{b: :b})

Similarly, moving an expression out the pattern may no longer preserve
its semantics. For example:

    match?([_ | _], [1, 2, 3])
    #=> true
    
    pattern = [_ | _]
    match?(pattern, [1, 2, 3])
    ** (CompileError) invalid use of _. _ can only be used inside patterns to ignore values and cannot be used in expressions. Make sure you are inside a pattern or change it accordingly

Another example is that a map as a pattern performs a subset match, but not
once assigned to a variable:

    match?(%{x: 1}, %{x: 1, y: 2})
    #=> true
    
    attrs = %{x: 1}
    match?(^attrs, %{x: 1, y: 2})
    #=> false

The pin operator will check if the values are equal, using `===/2`, while
patterns have their own rules when matching maps, lists, and so forth.
Such behavior is not specific to `match?/2`. The following code also
throws an exception:

    attrs = %{x: 1}
    ^attrs = %{x: 1, y: 2}
    #=> (MatchError) no match of right hand side value: %{x: 1, y: 2}


### max(first, second)

```elixir
@spec max(first, second) :: first | second when first: term(), second: term()
```

Returns the biggest of the two given terms according to
their structural comparison.

If the terms compare equal, the first one is returned.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Inlined by the compiler.

#### Examples

    iex> max(1, 2)
    2
    iex> max("a", "b")
    "b"


### min(first, second)

```elixir
@spec min(first, second) :: first | second when first: term(), second: term()
```

Returns the smallest of the two given terms according to
their structural comparison.

If the terms compare equal, the first one is returned.

This performs a structural comparison where all Elixir
terms can be compared with each other. See the ["Structural
comparison"](#module-structural-comparison) section
for more information.

Inlined by the compiler.

#### Examples

    iex> min(1, 2)
    1
    iex> min("foo", "bar")
    "bar"


### pop_in(path)
*(macro)* 


Pops a key from the nested structure via the given `path`.

This is similar to `pop_in/2`, except the path is extracted via
a macro rather than passing a list. For example:

    pop_in(opts[:foo][:bar])

Is equivalent to:

    pop_in(opts, [:foo, :bar])

Note that in order for this macro to work, the complete path must always
be visible by this macro. For more information about the supported path
expressions, please check `get_and_update_in/2` docs.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> pop_in(users["john"][:age])
    {27, %{"john" => %{}, "meg" => %{age: 23}}}
    
    iex> users = %{john: %{age: 27}, meg: %{age: 23}}
    iex> pop_in(users.john[:age])
    {27, %{john: %{}, meg: %{age: 23}}}

In case any entry returns `nil`, its key will be removed
and the deletion will be considered a success.


### pop_in(data, keys)

```elixir
@spec pop_in(data, [Access.get_and_update_fun(term(), data) | term(), ...]) ::
  {term(), data}
when data: Access.container()
```

Pops a key from the given nested structure.

Uses the `Access` protocol to traverse the structures
according to the given `keys`, unless the `key` is a
function. If the key is a function, it will be invoked
as specified in `get_and_update_in/3`.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> pop_in(users, ["john", :age])
    {27, %{"john" => %{}, "meg" => %{age: 23}}}

In case any entry returns `nil`, its key will be removed
and the deletion will be considered a success.

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> pop_in(users, ["jane", :age])
    {nil, %{"john" => %{age: 27}, "meg" => %{age: 23}}}


### put_elem(tuple, index, value)

```elixir
@spec put_elem(tuple(), non_neg_integer(), term()) :: tuple()
```

Puts `value` at the given zero-based `index` in `tuple`.

Inlined by the compiler.

#### Examples

    iex> tuple = {:foo, :bar, 3}
    iex> put_elem(tuple, 0, :baz)
    {:baz, :bar, 3}


### put_in(path, value)
*(macro)* 


Puts a value in a nested structure via the given `path`.

This is similar to `put_in/3`, except the path is extracted via
a macro rather than passing a list. For example:

    put_in(opts[:foo][:bar], :baz)

Is equivalent to:

    put_in(opts, [:foo, :bar], :baz)

This also works with nested structs and the `struct.path.to.value` way to specify
paths:

    put_in(struct.foo.bar, :baz)

Note that in order for this macro to work, the complete path must always
be visible by this macro. For more information about the supported path
expressions, please check `get_and_update_in/2` docs.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> put_in(users["john"][:age], 28)
    %{"john" => %{age: 28}, "meg" => %{age: 23}}
    
    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> put_in(users["john"].age, 28)
    %{"john" => %{age: 28}, "meg" => %{age: 23}}


### put_in(data, keys, value)

```elixir
@spec put_in(Access.t(), [term(), ...], term()) :: Access.t()
```

Puts a value in a nested structure.

Uses the `Access` module to traverse the structures
according to the given `keys`, unless the `key` is a
function. If the key is a function, it will be invoked
as specified in `get_and_update_in/3`.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> put_in(users, ["john", :age], 28)
    %{"john" => %{age: 28}, "meg" => %{age: 23}}

If any of the intermediate values are nil, it will raise:

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> put_in(users, ["jane", :age], "oops")
    ** (ArgumentError) could not put/update key :age on a nil value


### raise(message)
*(macro)* 


Raises an exception.

If `message` is a string, it raises a `RuntimeError` exception with it.

If `message` is an atom, it just calls `raise/2` with the atom as the first
argument and `[]` as the second one.

If `message` is an exception struct, it is raised as is.

If `message` is anything else, `raise` will fail with an `ArgumentError`
exception.

#### Examples

    iex> raise "oops"
    ** (RuntimeError) oops
    
    try do
      1 + :foo
    rescue
      x in [ArithmeticError] ->
        IO.puts("that was expected")
        raise x
    end


### raise(exception, attributes)
*(macro)* 


Raises an exception.

Calls the `exception/1` function on the given argument (which has to be a
module name like `ArgumentError` or `RuntimeError`) passing `attributes`
in order to retrieve the exception struct.

Any module that contains a call to the `defexception/1` macro automatically
implements the `c:Exception.exception/1` callback expected by `raise/2`.
For more information, see `defexception/1`.

#### Examples

    iex> raise(ArgumentError, "Sample")
    ** (ArgumentError) Sample


### reraise(message, stacktrace)
*(macro)* 


Raises an exception preserving a previous stacktrace.

Works like `raise/1` but does not generate a new stacktrace.

Note that `__STACKTRACE__` can be used inside catch/rescue
to retrieve the current stacktrace.

#### Examples

    try do
      raise "oops"
    rescue
      exception ->
        reraise exception, __STACKTRACE__
    end


### reraise(exception, attributes, stacktrace)
*(macro)* 


Raises an exception preserving a previous stacktrace.

`reraise/3` works like `reraise/2`, except it passes arguments to the
`exception/1` function as explained in `raise/2`.

#### Examples

    try do
      raise "oops"
    rescue
      exception ->
        reraise WrapperError, [exception: exception], __STACKTRACE__
    end


### send(dest, message)

```elixir
@spec send(dest :: Process.dest(), message) :: message when message: any()
```

Sends a message to the given `dest` and returns the message.

`dest` may be a remote or local PID, a local port, a locally
registered name, or a tuple in the form of `{registered_name, node}` for a
registered name at another node.

For additional documentation, see the [`!` operator Erlang
documentation](https://www.erlang.org/doc/reference_manual/expressions#send).

Inlined by the compiler.

#### Examples

    iex> send(self(), :hello)
    :hello


### sigil_C(term, modifiers)
*(macro)* 


Handles the sigil `~C` for charlists.

It returns a charlist without interpolations and without escape
characters.

A charlist is a list of integers where all the integers are valid code points.
The three expressions below are equivalent:

    ~C"foo\n"
    [?f, ?o, ?o, ?\\, ?n]
    [102, 111, 111, 92, 110]

In practice, charlists are mostly used in specific scenarios such as
interfacing with older Erlang libraries that do not accept binaries as arguments.

#### Examples

    iex> ~C(foo)
    ~c"foo"
    
    iex> ~C(f#{o}o)
    ~c"f\#{o}o"
    
    iex> ~C(foo\n)
    ~c"foo\\n"


### sigil_c(term, modifiers)
*(macro)* 


Handles the sigil `~c` for charlists.

It returns a charlist, unescaping characters and replacing interpolations.

A charlist is a list of integers where all the integers are valid code points.
The three expressions below are equivalent:

    ~c"foo"
    [?f, ?o, ?o]
    [102, 111, 111]

In practice, charlists are mostly used in specific scenarios such as
interfacing with older Erlang libraries that do not accept binaries as arguments.

#### Examples

    iex> ~c(foo)
    ~c"foo"
    
    iex> ~c(f#{:o}o)
    ~c"foo"
    
    iex> ~c(f\#{:o}o)
    ~c"f\#{:o}o"

The list is only printed as a `~c` sigil if all code points are within the
ASCII range:

    iex> ~c"hełło"
    [104, 101, 322, 322, 111]
    
    iex> [104, 101, 108, 108, 111]
    ~c"hello"

See `Inspect.Opts` for more information.


### sigil_D(date_string, modifiers)
*(macro)* 


Handles the sigil `~D` for dates.

By default, this sigil uses the built-in `Calendar.ISO`, which
requires dates to be written in the ISO8601 format:

    ~D[yyyy-mm-dd]

such as:

    ~D[2015-01-13]

If you are using alternative calendars, any representation can
be used as long as you follow the representation by a single space
and the calendar name:

    ~D[SOME-REPRESENTATION My.Alternative.Calendar]

The lower case `~d` variant does not exist as interpolation
and escape characters are not useful for date sigils.

More information on dates can be found in the `Date` module.

#### Examples

    iex> ~D[2015-01-13]
    ~D[2015-01-13]


### sigil_N(naive_datetime_string, modifiers)
*(macro)* 


Handles the sigil `~N` for naive date times.

By default, this sigil uses the built-in `Calendar.ISO`, which
requires naive date times to be written in the ISO8601 format:

    ~N[yyyy-mm-dd hh:mm:ss]
    ~N[yyyy-mm-dd hh:mm:ss.ssssss]
    ~N[yyyy-mm-ddThh:mm:ss.ssssss]

such as:

    ~N[2015-01-13 13:00:07]
    ~N[2015-01-13T13:00:07.123]

If you are using alternative calendars, any representation can
be used as long as you follow the representation by a single space
and the calendar name:

    ~N[SOME-REPRESENTATION My.Alternative.Calendar]

The lower case `~n` variant does not exist as interpolation
and escape characters are not useful for date time sigils.

More information on naive date times can be found in the
`NaiveDateTime` module.

#### Examples

    iex> ~N[2015-01-13 13:00:07]
    ~N[2015-01-13 13:00:07]
    iex> ~N[2015-01-13T13:00:07.001]
    ~N[2015-01-13 13:00:07.001]


### sigil_r(term, modifiers)
*(macro)* 


Handles the sigil `~r` for regular expressions.

It returns a regular expression pattern, unescaping characters and replacing
interpolations.

More information on regular expressions can be found in the `Regex` module.

#### Examples

    iex> Regex.match?(~r/foo/, "foo")
    true
    
    iex> Regex.match?(~r/a#{:b}c/, "abc")
    true

While the `~r` sigil allows parens and brackets to be used as delimiters,
it is preferred to use `"` or `/` to avoid escaping conflicts with reserved
regex characters.


### sigil_S(term, modifiers)
*(macro)* 


Handles the sigil `~S` for strings.

It returns a string without interpolations and without escape
characters.

#### Examples

    iex> ~S(foo)
    "foo"
    iex> ~S(f#{o}o)
    "f\#{o}o"
    iex> ~S(\o/)
    "\\o/"


### sigil_s(term, modifiers)
*(macro)* 


Handles the sigil `~s` for strings.

It returns a string as if it was a double quoted string, unescaping characters
and replacing interpolations.

#### Examples

    iex> ~s(foo)
    "foo"
    
    iex> ~s(f#{:o}o)
    "foo"
    
    iex> ~s(f\#{:o}o)
    "f\#{:o}o"


### sigil_T(time_string, modifiers)
*(macro)* 


Handles the sigil `~T` for times.

By default, this sigil uses the built-in `Calendar.ISO`, which
requires times to be written in the ISO8601 format:

    ~T[hh:mm:ss]
    ~T[hh:mm:ss.ssssss]

such as:

    ~T[13:00:07]
    ~T[13:00:07.123]

If you are using alternative calendars, any representation can
be used as long as you follow the representation by a single space
and the calendar name:

    ~T[SOME-REPRESENTATION My.Alternative.Calendar]

The lower case `~t` variant does not exist as interpolation
and escape characters are not useful for time sigils.

More information on times can be found in the `Time` module.

#### Examples

    iex> ~T[13:00:07]
    ~T[13:00:07]
    iex> ~T[13:00:07.001]
    ~T[13:00:07.001]


### sigil_U(datetime_string, modifiers)
*(since 1.9.0)* *(macro)* 


Handles the sigil `~U` to create a UTC `DateTime`.

By default, this sigil uses the built-in `Calendar.ISO`, which
requires UTC date times to be written in the ISO8601 format:

    ~U[yyyy-mm-dd hh:mm:ssZ]
    ~U[yyyy-mm-dd hh:mm:ss.ssssssZ]
    ~U[yyyy-mm-ddThh:mm:ss.ssssss+00:00]

such as:

    ~U[2015-01-13 13:00:07Z]
    ~U[2015-01-13T13:00:07.123+00:00]

If you are using alternative calendars, any representation can
be used as long as you follow the representation by a single space
and the calendar name:

    ~U[SOME-REPRESENTATION My.Alternative.Calendar]

The given `datetime_string` must include "Z" or "00:00" offset
which marks it as UTC, otherwise an error is raised.

The lower case `~u` variant does not exist as interpolation
and escape characters are not useful for date time sigils.

More information on date times can be found in the `DateTime` module.

#### Examples

    iex> ~U[2015-01-13 13:00:07Z]
    ~U[2015-01-13 13:00:07Z]
    iex> ~U[2015-01-13T13:00:07.001+00:00]
    ~U[2015-01-13 13:00:07.001Z]


### sigil_W(term, modifiers)
*(macro)* 


Handles the sigil `~W` for list of words.

It returns a list of "words" split by whitespace without interpolations
and without escape characters.

#### Modifiers

- `s`: words in the list are strings (default)
- `a`: words in the list are atoms
- `c`: words in the list are charlists

#### Examples

    iex> ~W(foo #{bar} baz)
    ["foo", "\#{bar}", "baz"]


### sigil_w(term, modifiers)
*(macro)* 


Handles the sigil `~w` for list of words.

It returns a list of "words" split by whitespace. Character unescaping and
interpolation happens for each word.

#### Modifiers

- `s`: words in the list are strings (default)
- `a`: words in the list are atoms
- `c`: words in the list are charlists

#### Examples

    iex> ~w(foo #{:bar} baz)
    ["foo", "bar", "baz"]
    
    iex> ~w(foo #{" bar baz "})
    ["foo", "bar", "baz"]
    
    iex> ~w(--source test/enum_test.exs)
    ["--source", "test/enum_test.exs"]
    
    iex> ~w(foo bar baz)a
    [:foo, :bar, :baz]
    
    iex> ~w(foo bar baz)c
    [~c"foo", ~c"bar", ~c"baz"]


### spawn(fun)

```elixir
@spec spawn((-&gt; any())) :: pid()
```

Spawns the given function and returns its PID.

Typically developers do not use the `spawn` functions, instead they use
abstractions such as `Task`, `GenServer` and `Agent`, built on top of
`spawn`, that spawns processes with more conveniences in terms of
introspection and debugging.

Check the `Process` module for more process-related functions.

The anonymous function receives 0 arguments, and may return any value.

Inlined by the compiler.

#### Examples

    current = self()
    child = spawn(fn -> send(current, {self(), 1 + 2}) end)
    
    receive do
      {^child, 3} -> IO.puts("Received 3 back")
    end


### spawn(module, fun, args)

```elixir
@spec spawn(module(), atom(), list()) :: pid()
```

Spawns the given function `fun` from the given `module` passing it the given
`args` and returns its PID.

Typically developers do not use the `spawn` functions, instead they use
abstractions such as `Task`, `GenServer` and `Agent`, built on top of
`spawn`, that spawns processes with more conveniences in terms of
introspection and debugging.

Check the `Process` module for more process-related functions.

Inlined by the compiler.

#### Examples

    spawn(SomeModule, :function, [1, 2, 3])


### spawn_link(fun)

```elixir
@spec spawn_link((-&gt; any())) :: pid()
```

Spawns the given function, links it to the current process, and returns its PID.

Typically developers do not use the `spawn` functions, instead they use
abstractions such as `Task`, `GenServer` and `Agent`, built on top of
`spawn`, that spawns processes with more conveniences in terms of
introspection and debugging.

Check the `Process` module for more process-related functions. For more
information on linking, check `Process.link/1`.

The anonymous function receives 0 arguments, and may return any value.

Inlined by the compiler.

#### Examples

    current = self()
    child = spawn_link(fn -> send(current, {self(), 1 + 2}) end)
    
    receive do
      {^child, 3} -> IO.puts("Received 3 back")
    end


### spawn_link(module, fun, args)

```elixir
@spec spawn_link(module(), atom(), list()) :: pid()
```

Spawns the given function `fun` from the given `module` passing it the given
`args`, links it to the current process, and returns its PID.

Typically developers do not use the `spawn` functions, instead they use
abstractions such as `Task`, `GenServer` and `Agent`, built on top of
`spawn`, that spawns processes with more conveniences in terms of
introspection and debugging.

Check the `Process` module for more process-related functions. For more
information on linking, check `Process.link/1`.

Inlined by the compiler.

#### Examples

    spawn_link(SomeModule, :function, [1, 2, 3])


### spawn_monitor(fun)

```elixir
@spec spawn_monitor((-&gt; any())) :: {pid(), reference()}
```

Spawns the given function, monitors it and returns its PID
and monitoring reference.

Typically developers do not use the `spawn` functions, instead they use
abstractions such as `Task`, `GenServer` and `Agent`, built on top of
`spawn`, that spawns processes with more conveniences in terms of
introspection and debugging.

Check the `Process` module for more process-related functions.

The anonymous function receives 0 arguments, and may return any value.

Inlined by the compiler.

#### Examples

    current = self()
    spawn_monitor(fn -> send(current, {self(), 1 + 2}) end)


### spawn_monitor(module, fun, args)

```elixir
@spec spawn_monitor(module(), atom(), list()) :: {pid(), reference()}
```

Spawns the given module and function passing the given args,
monitors it and returns its PID and monitoring reference.

Typically developers do not use the `spawn` functions, instead they use
abstractions such as `Task`, `GenServer` and `Agent`, built on top of
`spawn`, that spawns processes with more conveniences in terms of
introspection and debugging.

Check the `Process` module for more process-related functions.

Inlined by the compiler.

#### Examples

    spawn_monitor(SomeModule, :function, [1, 2, 3])


### struct(struct, fields \\ [])

```elixir
@spec struct(module() | struct(), Enumerable.t()) :: struct()
```

Creates and updates a struct.

The `struct` argument may be an atom (which defines `defstruct`)
or a `struct` itself. The second argument is any `Enumerable` that
emits two-element tuples (key-value pairs) during enumeration.

Keys in the `Enumerable` that don't exist in the struct are automatically
discarded. Note that keys must be atoms, as only atoms are allowed when
defining a struct. If there are duplicate keys in the `Enumerable`, the last
entry will be taken (same behavior as `Map.new/1`).

This function is useful for dynamically creating and updating structs, as
well as for converting maps to structs; in the latter case, just inserting
the appropriate `:__struct__` field into the map may not be enough and
`struct/2` should be used instead.

#### Examples

    defmodule User do
      defstruct name: "john"
    end
    
    struct(User)
    #=> %User{name: "john"}
    
    opts = [name: "meg"]
    user = struct(User, opts)
    #=> %User{name: "meg"}
    
    struct(user, unknown: "value")
    #=> %User{name: "meg"}
    
    struct(User, %{name: "meg"})
    #=> %User{name: "meg"}
    
    # String keys are ignored
    struct(User, %{"name" => "meg"})
    #=> %User{name: "john"}


### struct!(struct, fields \\ [])

```elixir
@spec struct!(module() | struct(), Enumerable.t()) :: struct()
```

Similar to `struct/2` but checks for key validity.

The function `struct!/2` emulates the compile time behavior
of structs. This means that:

- when building a struct, as in `struct!(SomeStruct, key: :value)`,
  it is equivalent to `%SomeStruct{key: :value}` and therefore this
  function will check if every given key-value belongs to the struct.
  If the struct is enforcing any key via `@enforce_keys`, those will
  be enforced as well;

- when updating a struct, as in `struct!(%SomeStruct{}, key: :value)`,
  it is equivalent to `%SomeStruct{struct | key: :value}` and therefore this
  function will check if every given key-value belongs to the struct.


### tap(value, fun)
*(since 1.12.0)* *(macro)* 


Pipes the first argument, `value`, into the second argument, a function `fun`,
and returns `value` itself.

Useful for running synchronous side effects in a pipeline, using the `|>/2` operator.

#### Examples

    iex> tap(1, fn x -> x + 1 end)
    1

Most commonly, this is used in pipelines, using the `|>/2` operator.
For example, let's suppose you want to inspect part of a data structure.
You could write:

    %{a: 1}
    |> Map.update!(:a, & &1 + 2)
    |> tap(&IO.inspect(&1.a))
    |> Map.update!(:a, & &1 * 2)


### then(value, fun)
*(since 1.12.0)* *(macro)* 


Pipes the first argument, `value`, into the second argument, a function `fun`,
and returns the result of calling `fun`.

In other words, it invokes the function `fun` with `value` as argument,
and returns its result.

This is most commonly used in pipelines, using the `|>/2` operator, allowing you
to pipe a value to a function outside of its first argument.

#### Examples

    iex> 1 |> then(fn x -> x * 2 end)
    2
    
    iex> 1 |> then(fn x -> Enum.drop(["a", "b", "c"], x) end)
    ["b", "c"]


### throw(term)

```elixir
@spec throw(term()) :: no_return()
```

A non-local return from a function.

Using `throw/1` is generally discouraged, as it allows a function
to escape from its regular execution flow, which can make the code
harder to read. Furthermore, all thrown values must be caught by
`try/catch`. See `try/1` for more information.

Inlined by the compiler.


### to_charlist(term)
*(macro)* 


Converts the given term to a charlist according to the `List.Chars` protocol.

#### Examples

    iex> to_charlist(:foo)
    ~c"foo"


### to_string(term)
*(macro)* 


Converts the argument to a string according to the
`String.Chars` protocol.

This is the function invoked when there is string interpolation.

#### Examples

    iex> to_string(:foo)
    "foo"


### to_timeout(duration)
*(since 1.17.0)* 
```elixir
@spec to_timeout([{unit, non_neg_integer()}] | timeout() | Duration.t()) :: timeout()
when unit: :week | :day | :hour | :minute | :second | :millisecond
```

Constructs a millisecond timeout from the given components, duration, or timeout.

This function is useful for constructing timeouts to use in functions that
expect `t:timeout/0` values (such as `Process.send_after/4` and many others).

#### Argument

The `duration` argument can be one of a `Duration`, a `t:timeout/0`, or a list
of components. Each of these is described below.

##### Passing `Duration`s

`t:Duration.t/0` structs can be converted to timeouts. The given duration must have
`year` and `month` fields set to `0`, since those cannot be reliably converted to
milliseconds (due to the varying number of days in a month and year).

Microseconds in durations are converted to milliseconds (through `System.convert_time_unit/3`).

##### Passing components

The `duration` argument can also be keyword list which can contain the following
keys, each appearing at most once with a non-negative integer value:

- `:week` - the number of weeks (a week is always 7 days)
- `:day` - the number of days (a day is always 24 hours)
- `:hour` - the number of hours
- `:minute` - the number of minutes
- `:second` - the number of seconds
- `:millisecond` - the number of milliseconds

The timeout is calculated as the sum of the components, each multiplied by
the corresponding factor.

##### Passing timeouts

You can also pass timeouts directly to this functions, that is, milliseconds or
the atom `:infinity`. In this case, this function just returns the given argument.

#### Examples

With a keyword list:

    iex> to_timeout(hour: 1, minute: 30)
    5400000

With a duration:

    iex> to_timeout(%Duration{hour: 1, minute: 30})
    5400000

With a timeout:

    iex> to_timeout(5400000)
    5400000
    iex> to_timeout(:infinity)
    :infinity


### unless(condition, clauses)
*(macro)* 

This macro is deprecated. Use if/2 instead.
Provides an `unless` macro.

This macro evaluates and returns the `do` block passed in as the second
argument if `condition` evaluates to a falsy value (`false` or `nil`).
Otherwise, it returns the value of the `else` block if present or `nil` if not.

See also `if/2`.

#### Examples

    iex> unless(Enum.empty?([]), do: "Hello")
    nil
    
    iex> unless(Enum.empty?([1, 2, 3]), do: "Hello")
    "Hello"
    
    iex> unless Enum.sum([2, 2]) == 5 do
    ...>   "Math still works"
    ...> else
    ...>   "Math is broken"
    ...> end
    "Math still works"


### update_in(path, fun)
*(macro)* 


Updates a nested structure via the given `path`.

This is similar to `update_in/3`, except the path is extracted via
a macro rather than passing a list. For example:

    update_in(opts[:foo][:bar], &(&1 + 1))

Is equivalent to:

    update_in(opts, [:foo, :bar], &(&1 + 1))

This also works with nested structs and the `struct.path.to.value` way to specify
paths:

    update_in(struct.foo.bar, &(&1 + 1))

Note that in order for this macro to work, the complete path must always
be visible by this macro. For more information about the supported path
expressions, please check `get_and_update_in/2` docs.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> update_in(users["john"][:age], &(&1 + 1))
    %{"john" => %{age: 28}, "meg" => %{age: 23}}
    
    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> update_in(users["john"].age, &(&1 + 1))
    %{"john" => %{age: 28}, "meg" => %{age: 23}}


### update_in(data, keys, fun)

```elixir
@spec update_in(Access.t(), [term(), ...], (term() -&gt; term())) :: Access.t()
```

Updates a key in a nested structure.

Uses the `Access` module to traverse the structures
according to the given `keys`, unless the `key` is a
function. If the key is a function, it will be invoked
as specified in `get_and_update_in/3`.

`data` is a nested structure (that is, a map, keyword
list, or struct that implements the `Access` behaviour).
The `fun` argument receives the value of `key` (or `nil`
if `key` is not present) and the result replaces the value
in the structure.

#### Examples

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> update_in(users, ["john", :age], &(&1 + 1))
    %{"john" => %{age: 28}, "meg" => %{age: 23}}

Note the current value given to the anonymous function may be `nil`.
If any of the intermediate values are nil, it will raise:

    iex> users = %{"john" => %{age: 27}, "meg" => %{age: 23}}
    iex> update_in(users, ["jane", :age], & &1 + 1)
    ** (ArgumentError) could not put/update key :age on a nil value


### use(module, opts \\ [])
*(macro)* 


Uses the given module in the current context.

When calling:

    use MyModule, some: :options

Elixir will invoke `MyModule.__using__/1` passing the second argument of
`use` as its argument. Since `__using__/1` is typically a macro, all
the usual macro rules apply, and its return value should be quoted code
that is then inserted where `use/2` is called.

> #### Code injection {: .warning}
> 
> `use MyModule` works as a **code-injection point** in the caller.
> Given the caller of `use MyModule` has little control over how the
> code is injected, `use/2` should be used with care. If you can,
> avoid use in favor of `import/2` or `alias/2` whenever possible.

#### Examples

For example, to write test cases using the `ExUnit` framework provided
with Elixir, a developer should `use` the `ExUnit.Case` module:

    defmodule AssertionTest do
      use ExUnit.Case, async: true
    
      test "always pass" do
        assert true
      end
    end

In this example, Elixir will call the `__using__/1` macro in the
`ExUnit.Case` module with the keyword list `[async: true]` as its
argument.

In other words, `use/2` translates to:

    defmodule AssertionTest do
      require ExUnit.Case
      ExUnit.Case.__using__(async: true)
    
      test "always pass" do
        assert true
      end
    end

where `ExUnit.Case` defines the `__using__/1` macro:

    defmodule ExUnit.Case do
      defmacro __using__(opts) do
        # do something with opts
        quote do
          # return some code to inject in the caller
        end
      end
    end

#### Best practices

`__using__/1` is typically used when there is a need to set some state
(via module attributes) or callbacks (like `@before_compile`, see the
documentation for `Module` for more information) into the caller.

`__using__/1` may also be used to alias, require, or import functionality
from different modules:

    defmodule MyModule do
      defmacro __using__(_opts) do
        quote do
          import MyModule.Foo
          import MyModule.Bar
          import MyModule.Baz
    
          alias MyModule.Repo
        end
      end
    end

However, do not provide `__using__/1` if all it does is to import,
alias or require the module itself. For example, avoid this:

    defmodule MyModule do
      defmacro __using__(_opts) do
        quote do
          import MyModule
        end
      end
    end

In such cases, developers should instead import or alias the module
directly, so that they can customize those as they wish,
without the indirection behind `use/2`. Developers must also avoid
defining functions inside `__using__/1`.

Given `use MyModule` can generate any code, it may not be easy for
developers to understand the impact of `use MyModule`.

For this reason, to provide guidance and clarity, we recommend developers
to include an admonition block in their `@moduledoc` that explains how
`use MyModule` impacts their code. As an example, the `GenServer` documentation
outlines:

> #### `use GenServer` {: .info}
> 
> When you `use GenServer`, the `GenServer` module will
> set `@behaviour GenServer` and define a `child_spec/1`
> function, so your module can be used as a child
> in a supervision tree.

This provides a quick summary of how using a module impacts the user code.
Keep in mind to only list changes made to the public API of the module.
For example, if `use MyModule` sets an internal attribute called
`@_my_module_info` and this attribute is never meant to be public,
it must not be listed.

For convenience, the markup notation to generate the admonition block
above is:

    > #### `use GenServer` {: .info}
    >
    > When you `use GenServer`, the GenServer module will
    > set `@behaviour GenServer` and define a `child_spec/1`
    > function, so your module can be used as a child
    > in a supervision tree.


### var!(var, context \\ nil)
*(macro)* 


Marks that the given variable should not be hygienized.

This macro expects a variable and it is typically invoked
inside `quote/2` to mark that a variable
should not be hygienized. See `quote/2` for more information.

#### Examples

    iex> Kernel.var!(example) = 1
    1
    iex> Kernel.var!(example)
    1


### left |&gt; right
*(macro)* 


Pipe operator.

This operator introduces the expression on the left-hand side as
the first argument to the function call on the right-hand side.

#### Examples

    iex> [1, [2], 3] |> List.flatten()
    [1, 2, 3]

The example above is the same as calling `List.flatten([1, [2], 3])`.

The `|>/2` operator is mostly useful when there is a desire to execute a series
of operations resembling a pipeline:

    iex> [1, [2], 3] |> List.flatten() |> Enum.map(fn x -> x * 2 end)
    [2, 4, 6]

In the example above, the list `[1, [2], 3]` is passed as the first argument
to the `List.flatten/1` function, then the flattened list is passed as the
first argument to the `Enum.map/2` function which doubles each element of the
list.

In other words, the expression above simply translates to:

    Enum.map(List.flatten([1, [2], 3]), fn x -> x * 2 end)

#### Pitfalls

There are two common pitfalls when using the pipe operator.

The first one is related to operator precedence. For example,
the following expression:

    String.graphemes "Hello" |> Enum.reverse

Translates to:

    String.graphemes("Hello" |> Enum.reverse())

which results in an error as the `Enumerable` protocol is not defined
for binaries. Adding explicit parentheses resolves the ambiguity:

    String.graphemes("Hello") |> Enum.reverse()

Or, even better:

    "Hello" |> String.graphemes() |> Enum.reverse()

The second limitation is that Elixir always pipes to a function
call. Therefore, to pipe into an anonymous function, you need to
invoke it:

    some_fun = &Regex.replace(~r/l/, &1, "L")
    "Hello" |> some_fun.()

Alternatively, you can use `then/2` for the same effect:

    some_fun = &Regex.replace(~r/l/, &1, "L")
    "Hello" |> then(some_fun)

`then/2` is most commonly used when you want to pipe to a function
but the value is expected outside of the first argument, such as
above. By replacing `some_fun` by its value, we get:

    "Hello" |> then(&Regex.replace(~r/l/, &1, "L"))


### left || right
*(macro)* 


Boolean "or" operator.

Provides a short-circuit operator that evaluates and returns the second
expression only if the first one does not evaluate to a truthy value (that is,
it is either `nil` or `false`). Returns the first expression otherwise.

Not allowed in guard clauses.

#### Examples

    iex> Enum.empty?([1]) || Enum.empty?([1])
    false
    
    iex> List.first([]) || true
    true
    
    iex> Enum.empty?([1]) || 1
    1
    
    iex> Enum.empty?([]) || throw(:bad)
    true

Note that, unlike `or/2`, this operator accepts any expression
as the first argument, not only booleans.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
