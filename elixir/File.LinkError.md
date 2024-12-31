# File.LinkError exception
(Elixir v1.18.0-dev)

An exception that is raised when linking a file fails.

The following fields of this exception are public and can be accessed freely:

- `:existing` (`t:Path.t/0`) - the existing file to link
- `:new` (`t:Path.t/0`) - the link destination
- `:reason` (`t:File.posix/0`) - the reason why the file could not be linked




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
