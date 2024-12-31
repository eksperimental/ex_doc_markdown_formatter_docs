# Protocol.UndefinedError exception
(Elixir v1.18.0-dev)

An exception raised when a protocol is not implemented for a given value.

The following fields of this exception are public and can be accessed freely:

- `:protocol` (`t:module/0`) - the protocol that is not implemented
- `:value` (`t:term/0`) - the value that does not implement the protocol

For example, this code:

    Enum.at("A string!", 0)

would raise the following exception:

    %Protocol.UndefinedError{
      protocol: Enumerable,
      value: "A string!",
      # ...
    }




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
