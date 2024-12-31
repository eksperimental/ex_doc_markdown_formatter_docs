# TokenMissingError exception
(Elixir v1.18.0-dev)

An exception raised when a token is missing when parsing code.

The following fields of this exceptions are public and can be accessed freely:

- `:file` (`t:Path.t/0` or `nil`) - the file where the error occurred, or `nil` if
  the error occurred in code that did not come from a file
- `:line` - the line for the opening delimiter
- `:column` - the column for the opening delimiter
- `:end_line` - the line for the end of the string
- `:end_column` - the column for the end of the string
- `:opening_delimiter` - an atom representing the opening delimiter
- `:expected_delimiter` - an atom representing the expected delimiter
- `:description` - a description of the missing token error




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
