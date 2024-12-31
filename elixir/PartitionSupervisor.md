# PartitionSupervisor 
(Elixir v1.18.0-dev)

A supervisor that starts multiple partitions of the same child.

Certain processes may become bottlenecks in large systems.
If those processes can have their state trivially partitioned,
in a way there is no dependency between them, then they can use
the `PartitionSupervisor` to create multiple isolated and
independent partitions.

Once the `PartitionSupervisor` starts, you can dispatch to its
children using `{:via, PartitionSupervisor, {name, key}}`, where
`name` is the name of the `PartitionSupervisor` and key is used
for routing.

This module was introduced in Elixir v1.14.0.

## Simple Example

Let's start with an example which is not useful per se, but shows how the
partitions are started and how messages are routed to them.

Here's a toy GenServer that simply collects the messages it's given.
It prints them for easy illustration.

    defmodule Collector do
      use GenServer
    
      def start_link(args) do
        GenServer.start_link(__MODULE__, args)
      end
    
      def init(args) do
        IO.inspect([__MODULE__, " got args ", args, " in ", self()])
        {:ok, _initial_state = []}
      end
    
      def collect(server, msg) do
        GenServer.call(server, {:collect, msg})
      end
    
      def handle_call({:collect, msg}, _from, state) do
        new_state = [msg | state]
        IO.inspect(["current messages:", new_state, " in process", self()])
        {:reply, :ok, new_state}
      end
    end

To run multiple of these, we can start them under a `PartitionSupervisor` by placing
this in our supervision tree:

    {PartitionSupervisor,
      child_spec: Collector.child_spec([some: :arg]),
      name: MyApp.PartitionSupervisor
    }

We can send messages to them using a "via tuple":

    # The key is used to route our message to a particular instance.
    key = 1
    Collector.collect({:via, PartitionSupervisor, {MyApp.PartitionSupervisor, key}}, :hi)
    # ["current messages:", [:hi], " in process", #PID<0.602.0>]
    :ok
    Collector.collect({:via, PartitionSupervisor, {MyApp.PartitionSupervisor, key}}, :ho)
    # ["current messages:", [:ho, :hi], " in process", #PID<0.602.0>]
    :ok
    
    # With a different key, the message will be routed to a different instance.
    key = 2
    Collector.collect({:via, PartitionSupervisor, {MyApp.PartitionSupervisor, key}}, :a)
    # ["current messages:", [:a], " in process", #PID<0.603.0>]
    :ok
    Collector.collect({:via, PartitionSupervisor, {MyApp.PartitionSupervisor, key}}, :b)
    # ["current messages:", [:b, :a], " in process", #PID<0.603.0>]
    :ok

Now let's move on to a useful example.

## `DynamicSupervisor` Example

The `DynamicSupervisor` is a single process responsible for starting
other processes. In some applications, the `DynamicSupervisor` may
become a bottleneck. To address this, you can start multiple instances
of the `DynamicSupervisor` through a `PartitionSupervisor`, and then
pick a "random" instance to start the child on.

Instead of starting a single `DynamicSupervisor`:

    children = [
      {DynamicSupervisor, name: MyApp.DynamicSupervisor}
    ]
    
    Supervisor.start_link(children, strategy: :one_for_one)

and starting children on that dynamic supervisor directly:

    DynamicSupervisor.start_child(MyApp.DynamicSupervisor, {Agent, fn -> %{} end})

You can start the dynamic supervisors under a `PartitionSupervisor`:

    children = [
      {PartitionSupervisor,
       child_spec: DynamicSupervisor,
       name: MyApp.DynamicSupervisors}
    ]
    
    Supervisor.start_link(children, strategy: :one_for_one)

and then:

    DynamicSupervisor.start_child(
      {:via, PartitionSupervisor, {MyApp.DynamicSupervisors, self()}},
      {Agent, fn -> %{} end}
    )

In the code above, we start a partition supervisor that will by default
start a dynamic supervisor for each core in your machine. Then, instead
of calling the `DynamicSupervisor` by name, you call it through the
partition supervisor using the `{:via, PartitionSupervisor, {name, key}}`
format. We picked `self()` as the routing key, which means each process
will be assigned one of the existing dynamic supervisors. See `start_link/1`
to see all options supported by the `PartitionSupervisor`.

## Implementation notes

The `PartitionSupervisor` uses either an ETS table or a `Registry` to
manage all of the partitions. Under the hood, the `PartitionSupervisor`
generates a child spec for each partition and then acts as a regular
supervisor. The ID of each child spec is the partition number.

For routing, two strategies are used. If `key` is an integer, it is routed
using `rem(abs(key), partitions)` where `partitions` is the number of
partitions. Otherwise it uses `:erlang.phash2(key, partitions)`.
The particular routing may change in the future, and therefore must not
be relied on. If you want to retrieve a particular PID for a certain key,
you can use `GenServer.whereis({:via, PartitionSupervisor, {name, key}})`.

## Types

### name()
*(since 1.14.0)* 
```elixir
@type name() :: atom() | {:via, module(), term()}
```

The name of the `PartitionSupervisor`.

## Functions

### count_children(supervisor)
*(since 1.14.0)* 
```elixir
@spec count_children(name()) :: %{
  specs: non_neg_integer(),
  active: non_neg_integer(),
  supervisors: non_neg_integer(),
  workers: non_neg_integer()
}
```

Returns a map containing count values for the supervisor.

The map contains the following keys:

- `:specs` - the number of partitions (children processes)

- `:active` - the count of all actively running child processes managed by
  this supervisor

- `:supervisors` - the count of all supervisors whether or not the child
  process is still alive

- `:workers` - the count of all workers, whether or not the child process
  is still alive

### partitions(name)
*(since 1.14.0)* 
```elixir
@spec partitions(name()) :: pos_integer()
```

Returns the number of partitions for the partition supervisor.

### resize!(name, partitions)
*(since 1.18.0)* 
```elixir
@spec resize!(name(), non_neg_integer()) :: non_neg_integer()
```

Resizes the number of partitions in the PartitionSupervisor.

This is done by starting or stopping a given number of
partitions in the supervisor. All of the child specifications
are kept in the `PartitionSupervisor` itself.

The final number of partitions cannot be less than zero and
cannot be more than the number of partitions the supervisor
started with.

### start_link(opts)
*(since 1.14.0)* 
```elixir
@spec start_link(keyword()) :: Supervisor.on_start()
```

Starts a partition supervisor with the given options.

This function is typically not invoked directly, instead it is invoked
when using a `PartitionSupervisor` as a child of another supervisor:

    children = [
      {PartitionSupervisor, child_spec: SomeChild, name: MyPartitionSupervisor}
    ]

If the supervisor is successfully spawned, this function returns
`{:ok, pid}`, where `pid` is the PID of the supervisor. If the given name
for the partition supervisor is already assigned to a process,
the function returns `{:error, {:already_started, pid}}`, where `pid`
is the PID of that process.

Note that a supervisor started with this function is linked to the parent
process and exits not only on crashes but also if the parent process exits
with `:normal` reason.

#### Options

- `:name` - an atom or via tuple representing the name of the partition
  supervisor (see `t:name/0`).

- `:child_spec` - the child spec to be used when starting the partitions.

- `:partitions` - a positive integer with the number of partitions.
  Defaults to `System.schedulers_online()` (typically the number of cores).

- `:strategy` - the restart strategy option, defaults to `:one_for_one`.
  You can learn more about strategies in the `Supervisor` module docs.

- `:max_restarts` - the maximum number of restarts allowed in
  a time frame. Defaults to `3`.

- `:max_seconds` - the time frame in which `:max_restarts` applies.
  Defaults to `5`.

- `:with_arguments` - a two-argument anonymous function that allows
  the partition to be given to the child starting function. See the
  `:with_arguments` section below.

#### `:with_arguments`

Sometimes you want each partition to know their partition assigned number.
This can be done with the `:with_arguments` option. This function receives
the value of the `:child_spec` option and an integer for the partition
number. It must return a new list of arguments that will be used to start the
partition process.

For example, most processes are started by calling `start_link(opts)`,
where `opts` is a keyword list. You could inject the partition into the
options given to the child:

    with_arguments: fn [opts], partition ->
      [Keyword.put(opts, :partition, partition)]
    end

### stop(supervisor, reason \\ :normal, timeout \\ :infinity)
*(since 1.14.0)* 
```elixir
@spec stop(name(), reason :: term(), timeout()) :: :ok
```

Synchronously stops the given partition supervisor with the given `reason`.

It returns `:ok` if the supervisor terminates with the given
reason. If it terminates with another reason, the call exits.

This function keeps OTP semantics regarding error reporting.
If the reason is any other than `:normal`, `:shutdown` or
`{:shutdown, _}`, an error report is logged.

### which_children(name)
*(since 1.14.0)* 
```elixir
@spec which_children(name()) :: [
  {integer(), pid() | :restarting, :worker | :supervisor, [module()] | :dynamic}
]
```

Returns a list with information about all children.

This function returns a list of tuples containing:

- `id` - the partition number

- `child` - the PID of the corresponding child process or the
  atom `:restarting` if the process is about to be restarted

- `type` - `:worker` or `:supervisor` as defined in the child
  specification

- `modules` - as defined in the child specification



---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
