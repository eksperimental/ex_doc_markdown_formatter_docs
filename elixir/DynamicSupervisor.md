# DynamicSupervisor behaviour
(Elixir v1.18.0-dev)

A supervisor optimized to only start children dynamically.

The `Supervisor` module was designed to handle mostly static children
that are started in the given order when the supervisor starts. A
`DynamicSupervisor` starts with no children. Instead, children are
started on demand via `start_child/2` and there is no ordering between
children. This allows the `DynamicSupervisor` to hold millions of
children by using efficient data structures and to execute certain
operations, such as shutting down, concurrently.

## Examples

A dynamic supervisor is started with no children and often a name:

    children = [
      {DynamicSupervisor, name: MyApp.DynamicSupervisor, strategy: :one_for_one}
    ]
    
    Supervisor.start_link(children, strategy: :one_for_one)

The options given in the child specification are documented in `start_link/1`.

Once the dynamic supervisor is running, we can use it to start children
on demand. Given this sample `GenServer`:

    defmodule Counter do
      use GenServer
    
      def start_link(initial) do
        GenServer.start_link(__MODULE__, initial)
      end
    
      def inc(pid) do
        GenServer.call(pid, :inc)
      end
    
      def init(initial) do
        {:ok, initial}
      end
    
      def handle_call(:inc, _, count) do
        {:reply, count, count + 1}
      end
    end

We can use `start_child/2` with a child specification to start a `Counter`
server:

    {:ok, counter1} = DynamicSupervisor.start_child(MyApp.DynamicSupervisor, {Counter, 0})
    Counter.inc(counter1)
    #=> 0
    
    {:ok, counter2} = DynamicSupervisor.start_child(MyApp.DynamicSupervisor, {Counter, 10})
    Counter.inc(counter2)
    #=> 10
    
    DynamicSupervisor.count_children(MyApp.DynamicSupervisor)
    #=> %{active: 2, specs: 2, supervisors: 0, workers: 2}

## Scalability and partitioning

The `DynamicSupervisor` is a single process responsible for starting
other processes. In some applications, the `DynamicSupervisor` may
become a bottleneck. To address this, you can start multiple instances
of the `DynamicSupervisor` and then pick a "random" instance to start
the child on.

Instead of:

    children = [
      {DynamicSupervisor, name: MyApp.DynamicSupervisor}
    ]

and:

    DynamicSupervisor.start_child(MyApp.DynamicSupervisor, {Counter, 0})

You can do this:

    children = [
      {PartitionSupervisor,
       child_spec: DynamicSupervisor,
       name: MyApp.DynamicSupervisors}
    ]

and then:

    DynamicSupervisor.start_child(
      {:via, PartitionSupervisor, {MyApp.DynamicSupervisors, self()}},
      {Counter, 0}
    )

In the code above, we start a partition supervisor that will by default
start a dynamic supervisor for each core in your machine. Then, instead
of calling the `DynamicSupervisor` by name, you call it through the
partition supervisor, using `self()` as the routing key. This means each
process will be assigned one of the existing dynamic supervisors.
Read the `PartitionSupervisor` docs for more information.

## Module-based supervisors

Similar to `Supervisor`, dynamic supervisors also support module-based
supervisors.

    defmodule MyApp.DynamicSupervisor do
      # Automatically defines child_spec/1
      use DynamicSupervisor
    
      def start_link(init_arg) do
        DynamicSupervisor.start_link(__MODULE__, init_arg, name: __MODULE__)
      end
    
      @impl true
      def init(_init_arg) do
        DynamicSupervisor.init(strategy: :one_for_one)
      end
    end

See the `Supervisor` docs for a discussion of when you may want to use
module-based supervisors. A `@doc` annotation immediately preceding
`use DynamicSupervisor` will be attached to the generated `child_spec/1`
function.

> #### `use DynamicSupervisor` {: .info}
> 
> When you `use DynamicSupervisor`, the `DynamicSupervisor` module will
> set `@behaviour DynamicSupervisor` and define a `child_spec/1`
> function, so your module can be used as a child in a supervision tree.

## Name registration

A supervisor is bound to the same name registration rules as a `GenServer`.
Read more about these rules in the documentation for `GenServer`.

## Migrating from Supervisor's :simple\_one\_for\_one

In case you were using the deprecated `:simple_one_for_one` strategy from
the `Supervisor` module, you can migrate to the `DynamicSupervisor` in
few steps.

Imagine the given "old" code:

    defmodule MySupervisor do
      use Supervisor
    
      def start_link(init_arg) do
        Supervisor.start_link(__MODULE__, init_arg, name: __MODULE__)
      end
    
      def start_child(foo, bar, baz) do
        # This will start child by calling MyWorker.start_link(init_arg, foo, bar, baz)
        Supervisor.start_child(__MODULE__, [foo, bar, baz])
      end
    
      @impl true
      def init(init_arg) do
        children = [
          # Or the deprecated: worker(MyWorker, [init_arg])
          %{id: MyWorker, start: {MyWorker, :start_link, [init_arg]}}
        ]
    
        Supervisor.init(children, strategy: :simple_one_for_one)
      end
    end

It can be upgraded to the DynamicSupervisor like this:

    defmodule MySupervisor do
      use DynamicSupervisor
    
      def start_link(init_arg) do
        DynamicSupervisor.start_link(__MODULE__, init_arg, name: __MODULE__)
      end
    
      def start_child(foo, bar, baz) do
        # If MyWorker is not using the new child specs, we need to pass a map:
        # spec = %{id: MyWorker, start: {MyWorker, :start_link, [foo, bar, baz]}}
        spec = {MyWorker, foo: foo, bar: bar, baz: baz}
        DynamicSupervisor.start_child(__MODULE__, spec)
      end
    
      @impl true
      def init(init_arg) do
        DynamicSupervisor.init(
          strategy: :one_for_one,
          extra_arguments: [init_arg]
        )
      end
    end

The difference is that the `DynamicSupervisor` expects the child specification
at the moment `start_child/2` is called, and no longer on the init callback.
If there are any initial arguments given on initialization, such as `[initial_arg]`,
it can be given in the `:extra_arguments` flag on `DynamicSupervisor.init/1`.


## Types

### init_option()

```elixir
@type init_option() ::
  {:strategy, strategy()}
  | {:max_restarts, non_neg_integer()}
  | {:max_seconds, pos_integer()}
  | {:max_children, non_neg_integer() | :infinity}
  | {:extra_arguments, [term()]}
```

Options given to `start_link/1` and `init/1` functions


### on_start_child()

```elixir
@type on_start_child() ::
  {:ok, pid()}
  | {:ok, pid(), info :: term()}
  | :ignore
  | {:error, {:already_started, pid()} | :max_children | term()}
```

Return values of `start_child` functions


### strategy()

```elixir
@type strategy() :: :one_for_one
```

Supported strategies


### sup_flags()

```elixir
@type sup_flags() :: %{
  strategy: strategy(),
  intensity: non_neg_integer(),
  period: pos_integer(),
  max_children: non_neg_integer() | :infinity,
  extra_arguments: [term()]
}
```

The supervisor flags returned on init


## Callbacks

### init(init_arg)

```elixir
@callback init(init_arg :: term()) :: {:ok, sup_flags()} | :ignore
```

Callback invoked to start the supervisor and during hot code upgrades.

Developers typically invoke `DynamicSupervisor.init/1` at the end of
their init callback to return the proper supervision flags.


## Functions

### child_spec(options)
*(since 1.6.1)* 


Returns a specification to start a dynamic supervisor under a supervisor.

It accepts the same options as `start_link/1`.

See `Supervisor` for more information about child specifications.


### count_children(supervisor)
*(since 1.6.0)* 
```elixir
@spec count_children(Supervisor.supervisor()) :: %{
  specs: non_neg_integer(),
  active: non_neg_integer(),
  supervisors: non_neg_integer(),
  workers: non_neg_integer()
}
```

Returns a map containing count values for the supervisor.

The map contains the following keys:

- `:specs` - the number of children processes

- `:active` - the count of all actively running child processes managed by
  this supervisor

- `:supervisors` - the count of all supervisors whether or not the child
  process is still alive

- `:workers` - the count of all workers, whether or not the child process
  is still alive


### init(options)
*(since 1.6.0)* 
```elixir
@spec init([init_option()]) :: {:ok, sup_flags()}
```

Receives a set of `options` that initializes a dynamic supervisor.

This is typically invoked at the end of the `c:init/1` callback of
module-based supervisors. See the "Module-based supervisors" section
in the module documentation for more information.

It accepts the same `options` as `start_link/1` (except for `:name`)
and it returns a tuple containing the supervisor options.

#### Examples

    def init(_arg) do
      DynamicSupervisor.init(max_children: 1000)
    end


### start_child(supervisor, child_spec)
*(since 1.6.0)* 
```elixir
@spec start_child(
  Supervisor.supervisor(),
  Supervisor.child_spec()
  | {module(), term()}
  | module()
  | (old_erlang_child_spec :: :supervisor.child_spec())
) :: on_start_child()
```

Dynamically adds a child specification to `supervisor` and starts that child.

`child_spec` should be a valid child specification as detailed in the
"Child specification" section of the documentation for `Supervisor`. The child
process will be started as defined in the child specification. Note that while
the `:id` field is still required in the spec, the value is ignored and
therefore does not need to be unique.

If the child process start function returns `{:ok, child}` or `{:ok, child, info}`, then child specification and PID are added to the supervisor and
this function returns the same value.

If the child process start function returns `:ignore`, then no child is added
to the supervision tree and this function returns `:ignore` too.

If the child process start function returns an error tuple or an erroneous
value, or if it fails, the child specification is discarded and this function
returns `{:error, error}` where `error` is the error or erroneous value
returned from child process start function, or failure reason if it fails.

If the supervisor already has N children in a way that N exceeds the amount
of `:max_children` set on the supervisor initialization (see `init/1`), then
this function returns `{:error, :max_children}`.


### start_link(options)
*(since 1.6.0)* 
```elixir
@spec start_link([init_option() | GenServer.option()]) :: Supervisor.on_start()
```

Starts a supervisor with the given options.

This function is typically not invoked directly, instead it is invoked
when using a `DynamicSupervisor` as a child of another supervisor:

    children = [
      {DynamicSupervisor, name: MySupervisor}
    ]

If the supervisor is successfully spawned, this function returns
`{:ok, pid}`, where `pid` is the PID of the supervisor. If the supervisor
is given a name and a process with the specified name already exists,
the function returns `{:error, {:already_started, pid}}`, where `pid`
is the PID of that process.

Note that a supervisor started with this function is linked to the parent
process and exits not only on crashes but also if the parent process exits
with `:normal` reason.

#### Options

- `:name` - registers the supervisor under the given name.
  The supported values are described under the "Name registration"
  section in the `GenServer` module docs.

- `:strategy` - the restart strategy option. The only supported
  value is `:one_for_one` which means that no other child is
  terminated if a child process terminates. You can learn more
  about strategies in the `Supervisor` module docs.

- `:max_restarts` - the maximum number of restarts allowed in
  a time frame. Defaults to `3`.

- `:max_seconds` - the time frame in which `:max_restarts` applies.
  Defaults to `5`.

- `:max_children` - the maximum amount of children to be running
  under this supervisor at the same time. When `:max_children` is
  exceeded, `start_child/2` returns `{:error, :max_children}`. Defaults
  to `:infinity`.

- `:extra_arguments` - arguments that are prepended to the arguments
  specified in the child spec given to `start_child/2`. Defaults to
  an empty list.

- Any of the standard [GenServer options](\`t:GenServer.option/0\`)


### start_link(module, init_arg, opts \\ [])
*(since 1.6.0)* 
```elixir
@spec start_link(module(), term(), [GenServer.option()]) :: Supervisor.on_start()
```

Starts a module-based supervisor process with the given `module` and `init_arg`.

To start the supervisor, the `c:init/1` callback will be invoked in the given
`module`, with `init_arg` as its argument. The `c:init/1` callback must return a
supervisor specification which can be created with the help of the `init/1`
function.

If the `c:init/1` callback returns `:ignore`, this function returns
`:ignore` as well and the supervisor terminates with reason `:normal`.
If it fails or returns an incorrect value, this function returns
`{:error, term}` where `term` is a term with information about the
error, and the supervisor terminates with reason `term`.

The `:name` option can also be given in order to register a supervisor
name, the supported values are described in the "Name registration"
section in the `GenServer` module docs.

If the supervisor is successfully spawned, this function returns
`{:ok, pid}`, where `pid` is the PID of the supervisor. If the supervisor
is given a name and a process with the specified name already exists,
the function returns `{:error, {:already_started, pid}}`, where `pid`
is the PID of that process.

Note that a supervisor started with this function is linked to the parent
process and exits not only on crashes but also if the parent process exits
with `:normal` reason.

#### Options

This function accepts any regular [`GenServer` options](\`t:GenServer.option/0\`).
Options specific to `DynamicSupervisor` must be returned from the `c:init/1`
callback.


### stop(supervisor, reason \\ :normal, timeout \\ :infinity)
*(since 1.7.0)* 
```elixir
@spec stop(Supervisor.supervisor(), reason :: term(), timeout()) :: :ok
```

Synchronously stops the given supervisor with the given `reason`.

It returns `:ok` if the supervisor terminates with the given
reason. If it terminates with another reason, the call exits.

This function keeps OTP semantics regarding error reporting.
If the reason is any other than `:normal`, `:shutdown` or
`{:shutdown, _}`, an error report is logged.


### terminate_child(supervisor, pid)
*(since 1.6.0)* 
```elixir
@spec terminate_child(Supervisor.supervisor(), pid()) :: :ok | {:error, :not_found}
```

Terminates the given child identified by `pid`.

If successful, this function returns `:ok`. If there is no process with
the given PID, this function returns `{:error, :not_found}`.


### which_children(supervisor)
*(since 1.6.0)* 
```elixir
@spec which_children(Supervisor.supervisor()) :: [
  {:undefined, pid() | :restarting, :worker | :supervisor,
   [module()] | :dynamic}
]
```

Returns a list with information about all children.

Note that calling this function when supervising a large number
of children under low memory conditions can cause an out of memory
exception.

This function returns a list of tuples containing:

- `id` - it is always `:undefined` for dynamic supervisors

- `child` - the PID of the corresponding child process or the
  atom `:restarting` if the process is about to be restarted

- `type` - `:worker` or `:supervisor` as defined in the child
  specification

- `modules` - as defined in the child specification




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
