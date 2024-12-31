# Config.Provider behaviour
(Elixir v1.18.0-dev)

Specifies a provider API that loads configuration during boot.

Config providers are typically used during releases to load
external configuration while the system boots. This is done
by starting the VM with the minimum amount of applications
running, then invoking all of the providers, and then
restarting the system. This requires a mutable configuration
file on disk, as the results of the providers are written to
the file system. For more information on runtime configuration,
see `mix release`.

## Multiple config files

One common use of config providers is to specify multiple
configuration files in a release. Elixir ships with one provider,
called `Config.Reader`, which is capable of handling Elixir's
built-in config files.

For example, imagine you want to list some basic configuration
on Mix's built-in `config/runtime.exs` file, but you also want
to support additional configuration files. To do so, you can add
this inside the `def project` portion of  your `mix.exs`:

    releases: [
      demo: [
        config_providers: [
          {Config.Reader, {:system, "RELEASE_ROOT", "/extra_config.exs"}}
        ]
      ]
    ]

You can place this `extra_config.exs` file in your release in
multiple ways:

1. If it is available on the host when assembling the release,
   you can place it on "rel/overlays/extra\_config.exs" and it
   will be automatically copied to the release root

2. If it is available on the target during deployment, you can
   simply copy it to the release root as a step in your deployment

Now once the system boots, it will load both `config/runtime.exs`
and `extra_config.exs` early in the boot process. You can learn
more options on `Config.Reader`.

## Custom config provider

You can also implement custom config providers, similar to how
`Config.Reader` works. For example, imagine you need to load
some configuration from a JSON file and load that into the system.
Said configuration provider would look like:

    defmodule JSONConfigProvider do
      @behaviour Config.Provider
    
      # Let's pass the path to the JSON file as config
      @impl true
      def init(path) when is_binary(path), do: path
    
      @impl true
      def load(config, path) do
        # We need to start any app we may depend on.
        {:ok, _} = Application.ensure_all_started(:jason)
    
        json = path |> File.read!() |> Jason.decode!()
    
        Config.Reader.merge(
          config,
          my_app: [
            some_value: json["my_app_some_value"],
            another_value: json["my_app_another_value"],
          ]
        )
      end
    end

Then, when specifying your release, you can specify the provider in
the release configuration:

    releases: [
      demo: [
        config_providers: [
          {JSONConfigProvider, "/etc/config.json"}
        ]
      ]
    ]

## Types

### config()

```elixir
@type config() :: keyword()
```



### config_path()

```elixir
@type config_path() :: {:system, binary(), binary()} | binary()
```

A path pointing to a configuration file.

Since configuration files are often accessed on target machines,
it can be expressed either as:

- a binary representing an absolute path

- a `{:system, system_var, path}` tuple where the config is the
  concatenation of the environment variable `system_var` with
  the given `path`

### state()

```elixir
@type state() :: term()
```



## Callbacks

### init(term)

```elixir
@callback init(term()) :: state()
```

Invoked when initializing a config provider.

A config provider is typically initialized on the machine
where the system is assembled and not on the target machine.
The `c:init/1` callback is useful to verify the arguments
given to the provider and prepare the state that will be
given to `c:load/2`.

Furthermore, because the state returned by `c:init/1` can
be written to text-based config files, it should be
restricted only to simple data types, such as integers,
strings, atoms, tuples, maps, and lists. Entries such as
PIDs, references, and functions cannot be serialized.

### load(config, state)

```elixir
@callback load(config(), state()) :: config()
```

Loads configuration (typically during system boot).

It receives the current `config` and the `state` returned by
`c:init/1`. Then, you typically read the extra configuration
from an external source and merge it into the received `config`.
Merging should be done with `Config.Reader.merge/2`, as it
performs deep merge. It should return the updated config.

Note that `c:load/2` is typically invoked very early in the
boot process, therefore if you need to use an application
in the provider, it is your responsibility to start it.

## Functions

### resolve_config_path!(path)
*(since 1.9.0)* 
```elixir
@spec resolve_config_path!(config_path()) :: binary()
```

Resolves a `t:config_path/0` to an actual path.

### validate_config_path!(path)
*(since 1.9.0)* 
```elixir
@spec validate_config_path!(config_path()) :: :ok
```

Validates a `t:config_path/0`.



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
