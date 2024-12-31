# MismatchedDelimiterError exception
(Elixir v1.18.0-dev)

An exception raised when a mismatched delimiter is found when parsing code.

For example:

- `[1, 2, 3}`
- `fn a -> )`

The following fields of this exceptions are public and can be accessed freely:

- `:file` (`t:Path.t/0` or `nil`) - the file where the error occurred, or `nil` if
  the error occurred in code that did not come from a file
- `:line` - the line for the opening delimiter
- `:column` - the column for the opening delimiter
- `:end_line` - the line for the mismatched closing delimiter
- `:end_column` - the column for the mismatched closing delimiter
- `:opening_delimiter` - an atom representing the opening delimiter
- `:closing_delimiter` - an atom representing the mismatched closing delimiter
- `:expected_delimiter` - an atom representing the closing delimiter
- `:description` - a description of the mismatched delimiter error



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
