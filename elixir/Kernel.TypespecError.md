# Kernel.TypespecError exception
(Elixir v1.18.0-dev)

An exception raised when there's an error in a typespec.

The following fields of this exceptions are public and can be accessed freely:

- `:file` (`t:Path.t/0` or `nil`) - the file where the error occurred, or `nil` if
  the error occurred in code that did not come from a file
- `:line` (`t:non_neg_integer/0`) - the line where the error occurred



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
