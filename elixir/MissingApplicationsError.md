# MissingApplicationsError exception
(Elixir v1.18.0-dev)

An exception that is raised when an application depends on one or more
missing applications.

This exception is used by Mix and other tools. It can also be used by library authors
when their library only requires an external application (like a dependency) for a subset
of features.

The fields of this exception are public. See `t:t/0`.

*Available since v1.18.0.*

## Examples

    unless Application.spec(:plug, :vsn) do
      raise MissingApplicationsError,
        description: "application :plug is required for testing Plug-related functionality",
        apps: [{:plug, "~> 1.0"}]
    end


## Types

### t()
*(since 1.18.0)* 
```elixir
@type t() :: %MissingApplicationsError{
  __exception__: true,
  apps: [{Application.app(), Version.requirement()}, ...],
  description: String.t()
}
```





---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
