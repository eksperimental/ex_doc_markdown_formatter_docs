# File.Error exception
(Elixir v1.18.0-dev)

An exception that is raised when a file operation fails.

The following fields of this exception are public and can be accessed freely:

- `:path` (`t:Path.t/0`) - the path of the file that caused the error
- `:reason` (`t:File.posix/0`) - the reason for the error



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
