# Config 
(Elixir v1.18.0-dev)

A simple keyword-based configuration API.

## Example

This module is most commonly used to define application configuration,
typically in `config/config.exs`:

    import Config
    
    config :some_app,
      key1: "value1",
      key2: "value2"
    
    import_config "#{config_env()}.exs"

`import Config` will import the functions `config/2`, `config/3`
`config_env/0`, `config_target/0`, and `import_config/1`
to help you manage your configuration.

`config/2` and `config/3` are used to define key-value configuration
for a given application. Once Mix starts, it will automatically
evaluate the configuration file and persist the configuration above
into `:some_app`'s application environment, which can be accessed in
as follows:

    "value1" = Application.fetch_env!(:some_app, :key1)

Finally, the line `import_config "#{config_env()}.exs"` will import
other config files based on the current configuration environment,
such as `config/dev.exs` and `config/test.exs`.

`Config` also provides a low-level API for evaluating and reading
configuration, under the `Config.Reader` module.

> #### Avoid application environment in libraries {: .info}
> 
> If you are writing a library to be used by other developers,
> it is generally recommended to avoid the application environment, as the
> application environment is effectively a global storage. Also note that
> the `config/config.exs` of a library is not evaluated when the library is
> used as a dependency, as configuration is always meant to configure the
> current project. For more information, see ["Using application configuration for
> libraries"](design-anti-patterns.md#using-application-configuration-for-libraries).

## Migrating from `use Mix.Config`

The `Config` module in Elixir was introduced in v1.9 as a replacement to
`use Mix.Config`, which was specific to Mix and has been deprecated.

You can leverage `Config` instead of `use Mix.Config` in three steps. The first
step is to replace `use Mix.Config` at the top of your config files by
`import Config`.

The second is to make sure your `import_config/1` calls do not have a
wildcard character. If so, you need to perform the wildcard lookup
manually. For example, if you did:

    import_config "../apps/*/config/config.exs"

It has to be replaced by:

    for config <- "../apps/*/config/config.exs" |> Path.expand(__DIR__) |> Path.wildcard() do
      import_config config
    end

The last step is to replace all `Mix.env()` calls in the config files with `config_env()`.

Keep in mind you must also avoid using `Mix.env()` inside your project files.
To check the environment at *runtime*, you may add a configuration key:

    # config.exs
    ...
    config :my_app, env: config_env()

Then, in other scripts and modules, you may get the environment with
`Application.fetch_env!/2`:

    # router.exs
    ...
    if Application.fetch_env!(:my_app, :env) == :prod do
      ...
    end

The only places where you may access functions from the `Mix` module are
the `mix.exs` file and inside custom Mix tasks, which are always within
the `Mix.Tasks` namespace.

## `config/runtime.exs`

For runtime configuration, you can use the `config/runtime.exs` file.
It is executed right before applications start in both Mix and releases
(assembled with `mix release`).

## Functions

### config(root_key, opts)
*(since 1.9.0)* 


Configures the given `root_key`.

Keyword lists are always deep-merged.

#### Examples

The given `opts` are merged into the existing configuration
for the given `root_key`. Conflicting keys are overridden by the
ones specified in `opts`, unless they are keywords, which are
deep merged recursively. For example, the application configuration
below

    config :logger,
      level: :warn,
      backends: [:console]
    
    config :logger,
      level: :info,
      truncate: 1024

will have a final configuration for `:logger` of:

    [level: :info, backends: [:console], truncate: 1024]

### config(root_key, key, opts)
*(since 1.9.0)* 


Configures the given `key` for the given `root_key`.

Keyword lists are always deep merged.

#### Examples

The given `opts` are merged into the existing values for `key`
in the given `root_key`. Conflicting keys are overridden by the
ones specified in `opts`, unless they are keywords, which are
deep merged recursively. For example, the application configuration
below

    config :ecto, Repo,
      log_level: :warn,
      adapter: Ecto.Adapters.Postgres,
      metadata: [read_only: true]
    
    config :ecto, Repo,
      log_level: :info,
      pool_size: 10,
      metadata: [replica: true]

will have a final value of the configuration for the `Repo`
key in the `:ecto` application of:

    Application.get_env(:ecto, Repo)
    #=> [
    #=>   log_level: :info,
    #=>   pool_size: 10,
    #=>   adapter: Ecto.Adapters.Postgres,
    #=>   metadata: [read_only: true, replica: true]
    #=> ]

### config_env()
*(since 1.11.0)* *(macro)* 


Returns the environment this configuration file is executed on.

In Mix projects this function returns the environment this configuration
file is executed on. In releases, the environment when `mix release` ran.

This is most often used to execute conditional code:

    if config_env() == :prod do
      config :my_app, :debug, false
    end

### config_target()
*(since 1.11.0)* *(macro)* 


Returns the target this configuration file is executed on.

This is most often used to execute conditional code:

    if config_target() == :host do
      config :my_app, :debug, false
    end

### import_config(file)
*(since 1.9.0)* *(macro)* 


Imports configuration from the given file.

In case the file doesn't exist, an error is raised.

If file is a relative, it will be expanded relatively to the
directory the current configuration file is in.

#### Examples

This is often used to emulate configuration across environments:

    import_config "#{config_env()}.exs"

Note, however, some configuration files, such as `config/runtime.exs`
does not support imports, as they are meant to be copied across
systems.

### read_config(root_key)
*(since 1.18.0)* 


Reads the configuration for the given root key.

This function only reads the configuration from a previous
`config/2` or `config/3` call. If `root_key` points to an
application, it does not read its actual application environment.
Its main use case is to make it easier to access and share
configuration values across files.

If the `root_key` was not configured, it returns `nil`.

#### Examples

    # In config/config.exs
    config :my_app, foo: :bar
    
    # In config/dev.exs
    config :another_app, foo: read_config(:my_app)[:foo] || raise "missing parent configuration"



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
