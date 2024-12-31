# Application behaviour
(Elixir v1.18.0-dev)

A module for working with applications and defining application callbacks.

Applications are the idiomatic way to package software in Erlang/OTP. To get
the idea, they are similar to the "library" concept common in other
programming languages, but with some additional characteristics.

An application is a component implementing some specific functionality, with a
standardized directory structure, configuration, and life cycle. Applications
are *loaded*, *started*, and *stopped*. Each application also has its own
environment, which provides a unified API for configuring each application.

Developers typically interact with the application environment and its
callback module. Therefore those will be the topics we will cover first
before jumping into details about the application resource file and life cycle.

## The application environment

Each application has its own environment. The environment is a keyword list
that maps atoms to terms. Note that this environment is unrelated to the
operating system environment.

By default, the environment of an application is an empty list. In a Mix
project's `mix.exs` file, you can set the `:env` key in `application/0`:

    def application do
      [env: [db_host: "localhost"]]
    end

Now, in your application, you can read this environment by using functions
such as `fetch_env!/2` and friends:

    defmodule MyApp.DBClient do
      def start_link() do
        SomeLib.DBClient.start_link(host: db_host())
      end
    
      defp db_host do
        Application.fetch_env!(:my_app, :db_host)
      end
    end

In Mix projects, the environment of the application and its dependencies can
be overridden via the `config/config.exs` and `config/runtime.exs` files. The
former is loaded at build-time, before your code compiles, and the latter at
runtime, just before your app starts. For example, someone using your application
can override its `:db_host` environment variable as follows:

    import Config
    config :my_app, :db_host, "db.local"

See the "Configuration" section in the `Mix` module for more information.
You can also change the application environment dynamically by using functions
such as `put_env/3` and `delete_env/2`.

> #### Application environment in libraries {: .info}
> 
> If you are writing a library to be used by other developers,
> it is generally recommended to avoid the application environment, as the
> application environment is effectively a global storage. For more information,
> read about this [anti-pattern](design-anti-patterns.md#using-application-configuration-for-libraries).

> #### Reading the environment of other applications {: .warning}
> 
> Each application is responsible for its own environment. Do not
> use the functions in this module for directly accessing or modifying
> the environment of other applications. Whenever you change the application
> environment, Elixir's build tool will only recompile the files that
> belong to that application. So if you read the application environment
> of another application, there is a chance you will be depending on
> outdated configuration, as your file won't be recompiled as it changes.

## Compile-time environment

In the previous example, we read the application environment at runtime:

    defmodule MyApp.DBClient do
      def start_link() do
        SomeLib.DBClient.start_link(host: db_host())
      end
    
      defp db_host do
        Application.fetch_env!(:my_app, :db_host)
      end
    end

In other words, the environment key `:db_host` for application `:my_app`
will only be read when `MyApp.DBClient` effectively starts. While reading
the application environment at runtime is the preferred approach, in some
rare occasions you may want to use the application environment to configure
the compilation of a certain project. However, if you try to access
`Application.fetch_env!/2` outside of a function:

    defmodule MyApp.DBClient do
      @db_host Application.fetch_env!(:my_app, :db_host)
    
      def start_link() do
        SomeLib.DBClient.start_link(host: @db_host)
      end
    end

You might see warnings and errors:

    warning: Application.fetch_env!/2 is discouraged in the module body,
    use Application.compile_env/3 instead
      iex:3: MyApp.DBClient
    
    ** (ArgumentError) could not fetch application environment :db_host
    for application :my_app because the application was not loaded nor
    configured

This happens because, when defining modules, the application environment
is not yet available. Luckily, the warning tells us how to solve this
issue, by using `Application.compile_env/3` instead:

    defmodule MyApp.DBClient do
      @db_host Application.compile_env(:my_app, :db_host, "db.local")
    
      def start_link() do
        SomeLib.DBClient.start_link(host: @db_host)
      end
    end

The difference here is that `compile_env` expects the default value to be
given as an argument, instead of using the `def application` function of
your `mix.exs`. Furthermore, by using `compile_env/3`, tools like Mix will
store the values used during compilation and compare the compilation values
with the runtime values whenever your system starts, raising an error in
case they differ.

In any case, compile-time environments should be avoided. Whenever possible,
reading the application environment at runtime should be the first choice.

## The application callback module

Applications can be loaded, started, and stopped. Generally, build tools
like Mix take care of starting an application and all of its dependencies
for you, but you can also do it manually by calling:

    {:ok, _} = Application.ensure_all_started(:some_app)

When an application starts, developers may configure a callback module
that executes custom code. Developers use this callback to start the
application supervision tree.

The first step to do so is to add a `:mod` key to the `application/0`
definition in your `mix.exs` file. It expects a tuple, with the application
callback module and start argument (commonly an empty list):

    def application do
      [mod: {MyApp, []}]
    end

The `MyApp` module given to `:mod` needs to implement the `Application` behaviour.
This can be done by putting `use Application` in that module and implementing the
`c:start/2` callback, for example:

    defmodule MyApp do
      use Application
    
      def start(_type, _args) do
        children = []
        Supervisor.start_link(children, strategy: :one_for_one)
      end
    end

> #### `use Application` {: .info}
> 
> When you `use Application`, the `Application` module will
> set `@behaviour Application` and define an overridable
> definition for the `c:stop/1` function, which is required
> by Erlang/OTP.

The `c:start/2` callback has to spawn and link a supervisor and return `{:ok, pid}` or `{:ok, pid, state}`, where `pid` is the PID of the supervisor, and
`state` is an optional application state. `args` is the second element of the
tuple given to the `:mod` option.

The `type` argument passed to `c:start/2` is usually `:normal` unless in a
distributed setup where application takeovers and failovers are configured.
Distributed applications are beyond the scope of this documentation.

When an application is shutting down, its `c:stop/1` callback is called after
the supervision tree has been stopped by the runtime. This callback allows the
application to do any final cleanup. The argument is the state returned by
`c:start/2`, if it did, or `[]` otherwise. The return value of `c:stop/1` is
ignored.

By using `Application`, modules get a default implementation of `c:stop/1`
that ignores its argument and returns `:ok`, but it can be overridden.

Application callback modules may also implement the optional callback
`c:prep_stop/1`. If present, `c:prep_stop/1` is invoked before the supervision
tree is terminated. Its argument is the state returned by `c:start/2`, if it did,
or `[]` otherwise, and its return value is passed to `c:stop/1`.

## The application resource file

In the sections above, we have configured an application in the
`application/0` section of the `mix.exs` file. Ultimately, Mix will use
this configuration to create an [*application resource
file*](https://www.erlang.org/doc/man/app), which is a file called
`APP_NAME.app`. For example, the application resource file of the OTP
application `ex_unit` is called `ex_unit.app`.

You can learn more about the generation of application resource files in
the documentation of `Mix.Tasks.Compile.App`, available as well by running
`mix help compile.app`.

## The application life cycle

### Loading applications

Applications are *loaded*, which means that the runtime finds and processes
their resource files:

    Application.load(:ex_unit)
    #=> :ok

When an application is loaded, the environment specified in its resource file
is merged with any overrides from config files.

Loading an application *does not* load its modules.

In practice, you rarely load applications by hand because that is part of the
start process, explained next.

### Starting applications

Applications are also *started*:

    Application.start(:ex_unit)
    #=> :ok

Once your application is compiled, running your system is a matter of starting
your current application and its dependencies. Differently from other languages,
Elixir does not have a `main` procedure that is responsible for starting your
system. Instead, you start one or more applications, each with their own
initialization and termination logic.

When an application is started, the `Application.load/1` is automatically
invoked if it hasn't been done yet. Then, it checks if the dependencies listed
in the `applications` key of the resource file are already started. Having at
least one dependency not started is an error condition. Functions like
`ensure_all_started/1` takes care of starting an application and all of its
dependencies for you.

If the application does not have a callback module configured, starting is
done at this point. Otherwise, its `c:start/2` callback is invoked. The PID of
the top-level supervisor returned by this function is stored by the runtime
for later use, and the returned application state is saved too, if any.

### Stopping applications

Started applications are, finally, *stopped*:

    Application.stop(:ex_unit)
    #=> :ok

Stopping an application without a callback module defined, is in practice a
no-op, except for some system tracing.

Stopping an application with a callback module has three steps:

1.  If present, invoke the optional callback `c:prep_stop/1`.
2.  Terminate the top-level supervisor.
3.  Invoke the required callback `c:stop/1`.

The arguments passed to the callbacks are related to the state optionally
returned by `c:start/2`, and are documented in the section about the callback
module above.

It is important to highlight that step 2 is a blocking one. Termination of a
supervisor triggers a recursive chain of children terminations, therefore
orderly shutting down all descendant processes. The `c:stop/1` callback is
invoked only after termination of the whole supervision tree.

Shutting down a live system cleanly can be done by calling `System.stop/1`. It
will shut down every application in the reverse order they were started.

By default, a SIGTERM from the operating system will automatically translate to
`System.stop/0`. You can also have more explicit control over operating system
signals via the `:os.set_signal/2` function.

## Tooling

The Mix build tool automates most of the application management tasks. For example,
`mix test` automatically starts your application dependencies and your application
itself before your test runs. `mix run --no-halt` boots your current project and
can be used to start a long running system. See `mix help run`.

Developers can also use `mix release` to build **releases**. Releases are able to
package all of your source code as well as the Erlang VM into a single directory.
Releases also give you explicit control over how each application is started and in
which order. They also provide a more streamlined mechanism for starting and
stopping systems, debugging, logging, as well as system monitoring.

Finally, Elixir provides tools such as escripts and archives, which are
different mechanisms for packaging your application. Those are typically used
when tools must be shared between developers and not as deployment options.
See `mix help archive.build` and `mix help escript.build` for more detail.

## Further information

For further details on applications please check the documentation of the
[`:application` Erlang module](\`:application\`), and the
[Applications](https://www.erlang.org/doc/design_principles/applications.html)
section of the [OTP Design Principles User's
Guide](https://www.erlang.org/doc/design_principles/users_guide.html).


## Types

### app()

```elixir
@type app() :: atom()
```



### application_key()

```elixir
@type application_key() ::
  :start_phases
  | :mod
  | :applications
  | :optional_applications
  | :included_applications
  | :registered
  | :maxT
  | :maxP
  | :modules
  | :vsn
  | :id
  | :description
```



### key()

```elixir
@type key() :: atom()
```



### restart_type()

```elixir
@type restart_type() :: :permanent | :transient | :temporary
```

Specifies the type of the application:

- `:permanent` - if `app` terminates, all other applications and the entire
  node are also terminated.

- `:transient` - if `app` terminates with `:normal` reason, it is reported
  but no other applications are terminated. If a transient application
  terminates abnormally, all other applications and the entire node are
  also terminated.

- `:temporary` - if `app` terminates, it is reported but no other
  applications are terminated (the default).

Note that it is always possible to stop an application explicitly by calling
`stop/1`. Regardless of the type of the application, no other applications will
be affected.

Note also that the `:transient` type is of little practical use, since when a
supervision tree terminates, the reason is set to `:shutdown`, not `:normal`.


### start_type()

```elixir
@type start_type() :: :normal | {:takeover, node()} | {:failover, node()}
```



### state()

```elixir
@type state() :: term()
```



### value()

```elixir
@type value() :: term()
```



## Callbacks

### config_change(changed, new, removed)
*(optional)* 
```elixir
@callback config_change(changed, new, removed) :: :ok
when changed: keyword(), new: keyword(), removed: [atom()]
```

Callback invoked after code upgrade, if the application environment
has changed.

`changed` is a keyword list of keys and their changed values in the
application environment. `new` is a keyword list with all new keys
and their values. `removed` is a list with all removed keys.


### prep_stop(state)
*(optional)* 
```elixir
@callback prep_stop(state()) :: state()
```

Called before stopping the application.

This function is called before the top-level supervisor is terminated. It
receives the state returned by `c:start/2`, if it did, or `[]` otherwise.
The return value is later passed to `c:stop/1`.


### start(start_type, start_args)

```elixir
@callback start(start_type(), start_args :: term()) ::
  {:ok, pid()} | {:ok, pid(), state()} | {:error, reason :: term()}
```

Called when an application is started.

This function is called when an application is started using
`Application.start/2` (and functions on top of that, such as
`Application.ensure_started/2`). This function should start the top-level
process of the application (which should be the top supervisor of the
application's supervision tree if the application follows the OTP design
principles around supervision).

`start_type` defines how the application is started:

- `:normal` - used if the startup is a normal startup or if the application
  is distributed and is started on the current node because of a failover
  from another node and the application specification key `:start_phases`
  is `:undefined`.
- `{:takeover, node}` - used if the application is distributed and is
  started on the current node because of a failover on the node `node`.
- `{:failover, node}` - used if the application is distributed and is
  started on the current node because of a failover on node `node`, and the
  application specification key `:start_phases` is not `:undefined`.

`start_args` are the arguments passed to the application in the `:mod`
specification key (for example, `mod: {MyApp, [:my_args]}`).

This function should either return `{:ok, pid}` or `{:ok, pid, state}` if
startup is successful. `pid` should be the PID of the top supervisor. `state`
can be an arbitrary term, and if omitted will default to `[]`; if the
application is later stopped, `state` is passed to the `stop/1` callback (see
the documentation for the `c:stop/1` callback for more information).

`use Application` provides no default implementation for the `start/2`
callback.


### start_phase(phase, start_type, phase_args)
*(optional)* 
```elixir
@callback start_phase(phase :: term(), start_type(), phase_args :: term()) ::
  :ok | {:error, reason :: term()}
```

Starts an application in synchronous phases.

This function is called after `start/2` finishes but before
`Application.start/2` returns. It will be called once for every start phase
defined in the application's (and any included applications') specification,
in the order they are listed in.


### stop(state)

```elixir
@callback stop(state()) :: term()
```

Called after an application has been stopped.

This function is called after an application has been stopped, i.e., after its
supervision tree has been stopped. It should do the opposite of what the
`c:start/2` callback did, and should perform any necessary cleanup. The return
value of this callback is ignored.

`state` is the state returned by `c:start/2`, if it did, or `[]` otherwise.
If the optional callback `c:prep_stop/1` is present, `state` is its return
value instead.

`use Application` defines a default implementation of this function which does
nothing and just returns `:ok`.


## Functions

### app_dir(app)

```elixir
@spec app_dir(app()) :: String.t()
```

Gets the directory for app.

This information is returned based on the code path. Here is an
example:

    File.mkdir_p!("foo/ebin")
    Code.prepend_path("foo/ebin")
    Application.app_dir(:foo)
    #=> "foo"

Even though the directory is empty and there is no `.app` file
it is considered the application directory based on the name
"foo/ebin". The name may contain a dash `-` which is considered
to be the app version and it is removed for the lookup purposes:

    File.mkdir_p!("bar-123/ebin")
    Code.prepend_path("bar-123/ebin")
    Application.app_dir(:bar)
    #=> "bar-123"

For more information on code paths, check the `Code` module in
Elixir and also Erlang's [`:code` module](\`:code\`).


### app_dir(app, path)

```elixir
@spec app_dir(app(), String.t() | [String.t()]) :: String.t()
```

Returns the given path inside `app_dir/1`.

If `path` is a string, then it will be used as the path inside `app_dir/1`. If
`path` is a list of strings, it will be joined (see `Path.join/1`) and the result
will be used as the path inside `app_dir/1`.

#### Examples

    File.mkdir_p!("foo/ebin")
    Code.prepend_path("foo/ebin")
    
    Application.app_dir(:foo, "my_path")
    #=> "foo/my_path"
    
    Application.app_dir(:foo, ["my", "nested", "path"])
    #=> "foo/my/nested/path"


### compile_env(app, key_or_path, default \\ nil)
*(since 1.10.0)* *(macro)* 
```elixir
@spec compile_env(app(), key() | list(), value()) :: value()
```

Reads the application environment at compilation time.

Similar to `get_env/3`, except it must be used to read values
at compile time. This allows Elixir to track when configuration
values change between compile time and runtime.

The first argument is the application name. The second argument
`key_or_path` is either an atom key or a path to traverse in
search of the configuration, starting with an atom key.

For example, imagine the following configuration:

    config :my_app, :key, [foo: [bar: :baz]]

We can access it during compile time as:

    Application.compile_env(:my_app, :key)
    #=> [foo: [bar: :baz]]
    
    Application.compile_env(:my_app, [:key, :foo])
    #=> [bar: :baz]
    
    Application.compile_env(:my_app, [:key, :foo, :bar])
    #=> :baz

A default value can also be given as third argument. If
any of the keys in the path along the way is missing, the
default value is used:

    Application.compile_env(:my_app, [:unknown, :foo, :bar], :default)
    #=> :default
    
    Application.compile_env(:my_app, [:key, :unknown, :bar], :default)
    #=> :default
    
    Application.compile_env(:my_app, [:key, :foo, :unknown], :default)
    #=> :default

Giving a path is useful to let Elixir know that only certain paths
in a large configuration are compile time dependent.


### compile_env(env, app, key_or_path, default)
*(since 1.14.0)* 
```elixir
@spec compile_env(Macro.Env.t(), app(), key() | list(), value()) :: value()
```

Reads the application environment at compilation time from a macro.

Typically, developers will use `compile_env/3`. This function must
only be invoked from macros which aim to read the compilation environment
dynamically.

It expects a `Macro.Env` as first argument, where the `Macro.Env` is
typically the `__CALLER__` in a macro. It raises if `Macro.Env` comes
from a function.


### compile_env!(app, key_or_path)
*(since 1.10.0)* *(macro)* 
```elixir
@spec compile_env!(app(), key() | list()) :: value()
```

Reads the application environment at compilation time or raises.

This is the same as `compile_env/3` but it raises an
`ArgumentError` if the configuration is not available.


### compile_env!(env, app, key_or_path)
*(since 1.14.0)* 
```elixir
@spec compile_env!(Macro.Env.t(), app(), key() | list()) :: value()
```

Reads the application environment at compilation time from a macro
or raises.

Typically, developers will use `compile_env!/2`. This function must
only be invoked from macros which aim to read the compilation environment
dynamically.

It expects a `Macro.Env` as first argument, where the `Macro.Env` is
typically the `__CALLER__` in a macro. It raises if `Macro.Env` comes
from a function.


### delete_env(app, key, opts \\ [])

```elixir
@spec delete_env(app(), key(), timeout: timeout(), persistent: boolean()) :: :ok
```

Deletes the `key` from the given `app` environment.

It receives the same options as `put_env/4`. Returns `:ok`.


### ensure_all_started(app_or_apps, type_or_opts \\ [])

```elixir
@spec ensure_all_started(app() | [app()],
  type: restart_type(),
  mode: :serial | :concurrent
) ::
  {:ok, [app()]} | {:error, term()}
@spec ensure_all_started(app() | [app()], restart_type()) ::
  {:ok, [app()]} | {:error, term()}
```

Ensures the given `app` or `apps` and their child applications are started.

The second argument is either the `t:restart_type/1` (for consistency with
`start/2`) or a keyword list.

#### Options

- `:type` - if the application should be started `:temporary` (default),
  `:permanent`, or `:transient`. See `t:restart_type/1` for more information.

- `:mode` - (since v1.15.0) if the applications should be started serially
  (`:serial`, default) or concurrently (`:concurrent`). This option requires
  Erlang/OTP 26+.


### ensure_loaded(app)
*(since 1.10.0)* 
```elixir
@spec ensure_loaded(app()) :: :ok | {:error, term()}
```

Ensures the given `app` is loaded.

Same as `load/1` but returns `:ok` if the application was already
loaded.


### ensure_started(app, type \\ :temporary)

```elixir
@spec ensure_started(app(), restart_type()) :: :ok | {:error, term()}
```

Ensures the given `app` is started with `t:restart_type/0`.

Same as `start/2` but returns `:ok` if the application was already
started.


### fetch_env(app, key)

```elixir
@spec fetch_env(app(), key()) :: {:ok, value()} | :error
```

Returns the value for `key` in `app`'s environment in a tuple.

If the configuration parameter does not exist, the function returns `:error`.

> #### Warning {: .warning}
> 
> You must use this function to read only your own application
> environment. Do not read the environment of other applications.

> #### Application environment in info
> 
> If you are writing a library to be used by other developers,
> it is generally recommended to avoid the application environment, as the
> application environment is effectively a global storage. For more information,
> read our [library guidelines](library-guidelines.md).


### fetch_env!(app, key)

```elixir
@spec fetch_env!(app(), key()) :: value()
```

Returns the value for `key` in `app`'s environment.

If the configuration parameter does not exist, raises `ArgumentError`.

> #### Warning {: .warning}
> 
> You must use this function to read only your own application
> environment. Do not read the environment of other applications.

> #### Application environment in info
> 
> If you are writing a library to be used by other developers,
> it is generally recommended to avoid the application environment, as the
> application environment is effectively a global storage. For more information,
> read our [library guidelines](library-guidelines.md).


### format_error(reason)

```elixir
@spec format_error(any()) :: String.t()
```

Formats the error reason returned by `start/2`,
`ensure_started/2`, `stop/1`, `load/1` and `unload/1`,
returns a string.


### get_all_env(app)

```elixir
@spec get_all_env(app()) :: [{key(), value()}]
```

Returns all key-value pairs for `app`.


### get_application(module)

```elixir
@spec get_application(atom()) :: atom() | nil
```

Gets the application for the given module.

The application is located by analyzing the spec
of all loaded applications. Returns `nil` if
the module is not listed in any application spec.


### get_env(app, key, default \\ nil)

```elixir
@spec get_env(app(), key(), value()) :: value()
```

Returns the value for `key` in `app`'s environment.

If the configuration parameter does not exist, the function returns the
`default` value.

> #### Warning {: .warning}
> 
> You must use this function to read only your own application
> environment. Do not read the environment of other applications.

#### Examples

`get_env/3` is commonly used to read the configuration of your OTP applications.
Since Mix configurations are commonly used to configure applications, we will use
this as a point of illustration.

Consider a new application `:my_app`. `:my_app` contains a database engine which
supports a pool of databases. The database engine needs to know the configuration for
each of those databases, and that configuration is supplied by key-value pairs in
environment of `:my_app`.

    config :my_app, Databases.RepoOne,
      # A database configuration
      ip: "localhost",
      port: 5433
    
    config :my_app, Databases.RepoTwo,
      # Another database configuration (for the same OTP app)
      ip: "localhost",
      port: 20717
    
    config :my_app, my_app_databases: [Databases.RepoOne, Databases.RepoTwo]

Our database engine used by `:my_app` needs to know what databases exist, and
what the database configurations are. The database engine can make a call to
`Application.get_env(:my_app, :my_app_databases, [])` to retrieve the list of
databases (specified by module names).

The engine can then traverse each repository in the list and call
`Application.get_env(:my_app, Databases.RepoOne)` and so forth to retrieve the
configuration of each one. In this case, each configuration will be a keyword
list, so you can use the functions in the `Keyword` module or even the `Access`
module to traverse it, for example:

    config = Application.get_env(:my_app, Databases.RepoOne)
    config[:ip]


### load(app)

```elixir
@spec load(app()) :: :ok | {:error, term()}
```

Loads the given `app`.

In order to be loaded, an `.app` file must be in the load paths.
All `:included_applications` will also be loaded.

Loading the application does not start it nor load its modules, but
it does load its environment.


### loaded_applications()

```elixir
@spec loaded_applications() :: [{app(), description :: charlist(), vsn :: charlist()}]
```

Returns a list with information about the applications which have been loaded.


### put_all_env(config, opts \\ [])
*(since 1.9.0)* 
```elixir
@spec put_all_env([{app(), [{key(), value()}]}],
  timeout: timeout(),
  persistent: boolean()
) :: :ok
```

Puts the environment for multiple applications at the same time.

The given config should not:

- have the same application listed more than once
- have the same key inside the same application listed more than once

If those conditions are not met, this function will raise.

This function receives the same options as `put_env/4`. Returns `:ok`.

#### Examples

    Application.put_all_env(
      my_app: [
        key: :value,
        another_key: :another_value
      ],
      another_app: [
        key: :value
      ]
    )


### put_env(app, key, value, opts \\ [])

```elixir
@spec put_env(app(), key(), value(), timeout: timeout(), persistent: boolean()) :: :ok
```

Puts the `value` in `key` for the given `app`.

> #### Compile environment {: .warning}
> 
> Do not use this function to change environment variables read
> via `Application.compile_env/2`. The compile environment must
> be exclusively set before compilation, in your config files.

#### Options

- `:timeout` - the timeout for the change (defaults to `5_000` milliseconds)
- `:persistent` - persists the given value on application load and reloads

If `put_env/4` is called before the application is loaded, the application
environment values specified in the `.app` file will override the ones
previously set.

The `:persistent` option can be set to `true` when there is a need to guarantee
parameters set with this function will not be overridden by the ones defined
in the application resource file on load. This means persistent values will
stick after the application is loaded and also on application reload.


### spec(app)

```elixir
@spec spec(app()) :: [{application_key(), value()}] | nil
```

Returns the spec for `app`.

The following keys are returned:

- `:description`
- `:id`
- `:vsn`
- `:modules`
- `:maxP`
- `:maxT`
- `:registered`
- `:included_applications`
- `:optional_applications`
- `:applications`
- `:mod`
- `:start_phases`

For a description of all fields, see [Erlang's application
specification](https://www.erlang.org/doc/man/app).

Note the environment is not returned as it can be accessed via
`fetch_env/2`. Returns `nil` if the application is not loaded.


### spec(app, key)

```elixir
@spec spec(app(), application_key()) :: value() | nil
```

Returns the value for `key` in `app`'s specification.

See `spec/1` for the supported keys. If the given
specification parameter does not exist, this function
will raise. Returns `nil` if the application is not loaded.


### start(app, type \\ :temporary)

```elixir
@spec start(app(), restart_type()) :: :ok | {:error, term()}
```

Starts the given `app` with `t:restart_type/0`.

If the `app` is not loaded, the application will first be loaded using `load/1`.
Any included application, defined in the `:included_applications` key of the
`.app` file will also be loaded, but they won't be started.

Furthermore, all applications listed in the `:applications` key must be explicitly
started before this application is. If not, `{:error, {:not_started, app}}` is
returned, where `app` is the name of the missing application.

In case you want to automatically load **and start** all of `app`'s dependencies,
see `ensure_all_started/2`.


### started_applications(timeout \\ 5000)

```elixir
@spec started_applications(timeout()) :: [
  {app(), description :: charlist(), vsn :: charlist()}
]
```

Returns a list with information about the applications which are currently running.


### stop(app)

```elixir
@spec stop(app()) :: :ok | {:error, term()}
```

Stops the given `app`.

When stopped, the application is still loaded.


### unload(app)

```elixir
@spec unload(app()) :: :ok | {:error, term()}
```

Unloads the given `app`.

It will also unload all `:included_applications`.
Note that the function does not purge the application modules.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").