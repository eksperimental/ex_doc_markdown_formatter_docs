# Behaviour 
(Elixir v1.18.0-dev)

This module is deprecated. Use @callback and @macrocallback attributes instead.
Mechanism for handling behaviours.

This module is deprecated. Instead of `defcallback/1` and
`defmacrocallback/1`, the `@callback` and `@macrocallback`
module attributes can be used respectively. See the
documentation for `Module` for more information on these
attributes.

Instead of `MyModule.__behaviour__(:callbacks)`,
`MyModule.behaviour_info(:callbacks)` can be used. `behaviour_info/1`
is documented in `Module`.

## Functions

### defcallback(spec)
*(macro)* 

**This macro is deprecated. Use the @callback module attribute instead.**

Defines a function callback according to the given type specification.

### defmacrocallback(spec)
*(macro)* 

**This macro is deprecated. Use the @macrocallback module attribute instead.**

Defines a macro callback according to the given type specification.



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
