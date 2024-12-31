# Config.Reader 
(Elixir v1.18.0-dev)

API for reading config files defined with `Config`.

## As a provider

`Config.Reader` can also be used as a `Config.Provider`. A config
provider is used during releases to customize how applications are
configured. When used as a provider, it expects a single argument:
the configuration path (as outlined in `t:Config.Provider.config_path/0`)
for the file to be read and loaded during the system boot.

For example, if you expect the target system to have a config file
in an absolute path, you can add this inside the `def project` portion
of  your `mix.exs`:

    releases: [
      demo: [
        config_providers: [
          {Config.Reader, "/etc/config.exs"}
        ]
      ]
    ]

Or if you want to read a custom path inside the release:

    config_providers: [{Config.Reader, {:system, "RELEASE_ROOT", "/config.exs"}}]

You can also pass a keyword list of options to the reader,
where the `:path` is a required key:

    config_providers: [
      {Config.Reader,
       path: "/etc/config.exs",
       env: :prod,
       imports: :disabled}
    ]

Remember Mix already loads `config/runtime.exs` by default.
For more examples and scenarios, see the `Config.Provider` module.


## Functions

### eval!(file, contents, opts \\ [])
*(since 1.11.0)* 
```elixir
@spec eval!(Path.t(), binary(), keyword()) :: keyword()
```

Evaluates the configuration `contents` for the given `file`.

Accepts the same options as `read!/2`.


### merge(config1, config2)
*(since 1.9.0)* 
```elixir
@spec merge(keyword(), keyword()) :: keyword()
```

Merges two configurations.

The configurations are merged together with the values in
the second one having higher preference than the first in
case of conflicts. In case both values are set to keyword
lists, it deep merges them.

#### Examples

    iex> Config.Reader.merge([app: [k: :v1]], [app: [k: :v2]])
    [app: [k: :v2]]
    
    iex> Config.Reader.merge([app: [k: [v1: 1, v2: 2]]], [app: [k: [v2: :a, v3: :b]]])
    [app: [k: [v1: 1, v2: :a, v3: :b]]]
    
    iex> Config.Reader.merge([app1: []], [app2: []])
    [app1: [], app2: []]


### read!(file, opts \\ [])
*(since 1.9.0)* 
```elixir
@spec read!(
  Path.t(),
  keyword()
) :: keyword()
```

Reads the configuration file.

#### Options

- `:imports` - a list of already imported paths or `:disabled`
  to disable imports

- `:env` - the environment the configuration file runs on.
  See `Config.config_env/0` for sample usage

- `:target` - the target the configuration file runs on.
  See `Config.config_target/0` for sample usage


### read_imports!(file, opts \\ [])
*(since 1.9.0)* 
```elixir
@spec read_imports!(
  Path.t(),
  keyword()
) :: {keyword(), [Path.t()]}
```

Reads the given configuration file and returns the configuration
with its imports.

Accepts the same options as `read!/2`. Although note the `:imports`
option cannot be disabled in `read_imports!/2`.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
