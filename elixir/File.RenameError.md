# File.RenameError exception
(Elixir v1.18.0-dev)

An exception that is raised when renaming a file fails.

The following fields of this exception are public and can be accessed freely:

- `:source` (`t:Path.t/0`) - the source path
- `:destination` (`t:Path.t/0`) - the destination path
- `:reason` (`t:File.posix/0`) - the reason why the file could not be renamed




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
