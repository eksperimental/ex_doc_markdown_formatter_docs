# Dict 
(Elixir v1.18.0-dev)

This module is deprecated. Use Map or Keyword modules instead.
Generic API for dictionaries.

If you need a general dictionary, use the `Map` module.
If you need to manipulate keyword lists, use `Keyword`.

To convert maps into keywords and vice-versa, use the
`new` function in the respective modules.


## Types

### key()

```elixir
@type key() :: any()
```



### t()

```elixir
@type t() :: list() | map()
```



### value()

```elixir
@type value() :: any()
```



## Functions

### delete(dict, key)

```elixir
@spec delete(t(), key()) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### drop(dict, keys)

```elixir
@spec drop(t(), [key()]) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### empty(dict)

```elixir
@spec empty(t()) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### equal?(dict1, dict2)

```elixir
@spec equal?(t(), t()) :: boolean()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### fetch(dict, key)

```elixir
@spec fetch(t(), key()) :: value()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### fetch!(dict, key)

```elixir
@spec fetch!(t(), key()) :: value()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### get(dict, key, default \\ nil)

```elixir
@spec get(t(), key(), value()) :: value()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### get_and_update(dict, key, fun)

```elixir
@spec get_and_update(t(), key(), (value() -&gt; {value(), value()})) :: {value(), t()}
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### get_lazy(dict, key, fun)

```elixir
@spec get_lazy(t(), key(), (-&gt; value())) :: value()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### has_key?(dict, key)

```elixir
@spec has_key?(t(), key()) :: boolean()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### keys(dict)

```elixir
@spec keys(t()) :: [key()]
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### merge(dict1, dict2)

```elixir
@spec merge(t(), t()) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### merge(dict1, dict2, fun)

```elixir
@spec merge(t(), t(), (key(), value(), value() -&gt; value())) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### pop(dict, key, default \\ nil)

```elixir
@spec pop(t(), key(), value()) :: {value(), t()}
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### pop_lazy(dict, key, fun)

```elixir
@spec pop_lazy(t(), key(), (-&gt; value())) :: {value(), t()}
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### put(dict, key, val)

```elixir
@spec put(t(), key(), value()) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### put_new(dict, key, val)

```elixir
@spec put_new(t(), key(), value()) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### put_new_lazy(dict, key, fun)

```elixir
@spec put_new_lazy(t(), key(), (-&gt; value())) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### size(dict)

```elixir
@spec size(t()) :: non_neg_integer()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### split(dict, keys)

```elixir
@spec split(t(), [key()]) :: {t(), t()}
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### take(dict, keys)

```elixir
@spec take(t(), [key()]) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### to_list(dict)

```elixir
@spec to_list(t()) :: list()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### update(dict, key, default, fun)

```elixir
@spec update(t(), key(), value(), (value() -&gt; value())) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### update!(dict, key, fun)

```elixir
@spec update!(t(), key(), (value() -&gt; value())) :: t()
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.


### values(dict)

```elixir
@spec values(t()) :: [value()]
```
This function is deprecated. Use the Map module for working with maps or the Keyword module for working with keyword lists.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
