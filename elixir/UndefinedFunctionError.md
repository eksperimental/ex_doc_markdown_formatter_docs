# UndefinedFunctionError exception
(Elixir v1.18.0-dev)

An exception raised when a function is invoked that is not defined.

The following fields of this exception are public and can be accessed freely:

- `:module` (`t:module/0`) - the module name
- `:function` (`t:atom/0`) - the function name
- `:arity` (`t:non_neg_integer/0`) - the arity of the function

For example, if you try to call `MyMod.non_existing_fun("hello", 1)`,
the error would look like:

    %UndefinedFunctionError{
      module: MyMod,
      function: :non_existing_fun,
      arity: 2,
      # Other private fields...
    }



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
