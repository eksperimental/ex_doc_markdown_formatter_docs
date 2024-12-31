# SyntaxError exception
(Elixir v1.18.0-dev)

An exception raised when there's a syntax error when parsing code.

The following fields of this exceptions are public and can be accessed freely:

- `:file` (`t:Path.t/0` or `nil`) - the file where the error occurred, or `nil` if
  the error occurred in code that did not come from a file
- `:line` - the line where the error occurred
- `:column` - the column where the error occurred
- `:description` - a description of the syntax error




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
