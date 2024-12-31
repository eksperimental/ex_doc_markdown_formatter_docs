# Node 
(Elixir v1.18.0-dev)

Functions related to VM nodes.

Some of the functions in this module are inlined by the compiler,
similar to functions in the `Kernel` module and they are explicitly
marked in their docs as "inlined by the compiler". For more information
about inlined functions, check out the `Kernel` module.


## Types

### state()

```elixir
@type state() :: :visible | :hidden | :connected | :this | :known
```



### t()

```elixir
@type t() :: node()
```



## Functions

### alive?()

```elixir
@spec alive?() :: boolean()
```

Returns `true` if the local node is alive.

That is, if the node can be part of a distributed system.


### connect(node)

```elixir
@spec connect(t()) :: boolean() | :ignored
```

Establishes a connection to `node`.

Returns `true` if successful, `false` if not, and the atom
`:ignored` if the local node is not alive.

For more information, see `:net_kernel.connect_node/1`.


### disconnect(node)

```elixir
@spec disconnect(t()) :: boolean() | :ignored
```

Forces the disconnection of a node.

This will appear to the `node` as if the local node has crashed.
This function is mainly used in the Erlang network authentication
protocols. Returns `true` if disconnection succeeds, otherwise `false`.
If the local node is not alive, the function returns `:ignored`.

For more information, see `:erlang.disconnect_node/1`.


### get_cookie()

```elixir
@spec get_cookie() :: atom()
```

Returns the magic cookie of the local node.

Returns the cookie if the node is alive, otherwise `:nocookie`.


### list()

```elixir
@spec list() :: [t()]
```

Returns a list of all visible nodes in the system, excluding
the local node.

Same as `list(:visible)`.

Inlined by the compiler.


### list(args)

```elixir
@spec list(state() | [state()]) :: [t()]
```

Returns a list of nodes according to argument given.

The result returned when the argument is a list, is the list of nodes
satisfying the disjunction(s) of the list elements.

For more information, see `:erlang.nodes/1`.

Inlined by the compiler.


### monitor(node, flag)

```elixir
@spec monitor(t(), boolean()) :: true
```

Monitors the status of the node.

If `flag` is `true`, monitoring is turned on.
If `flag` is `false`, monitoring is turned off.

For more information, see `:erlang.monitor_node/2`.

For monitoring status changes of all nodes, see `:net_kernel.monitor_nodes/2`.


### monitor(node, flag, options)

```elixir
@spec monitor(t(), boolean(), [:allow_passive_connect]) :: true
```

Behaves as `monitor/2` except that it allows an extra
option to be given, namely `:allow_passive_connect`.

For more information, see `:erlang.monitor_node/3`.

For monitoring status changes of all nodes, see `:net_kernel.monitor_nodes/2`.


### ping(node)

```elixir
@spec ping(t()) :: :pong | :pang
```

Tries to set up a connection to node.

Returns `:pang` if it fails, or `:pong` if it is successful.

#### Examples

    iex> Node.ping(:unknown_node)
    :pang


### self()

```elixir
@spec self() :: t()
```

Returns the current node.

It returns the same as the built-in `node()`.


### set_cookie(node \\ Node.self(), cookie)

```elixir
@spec set_cookie(t(), atom()) :: true
```

Sets the magic cookie of `node` to the atom `cookie`.

The default node is `Node.self/0`, the local node. If `node` is the local node,
the function also sets the cookie of all other unknown nodes to `cookie`.

This function will raise `FunctionClauseError` if the given `node` is not alive.


### spawn(node, fun)

```elixir
@spec spawn(t(), (-&gt; any())) :: pid()
```

Returns the PID of a new process started by the application of `fun`
on `node`. If `node` does not exist, a useless PID is returned.

For the list of available options, see `:erlang.spawn/2`.

Inlined by the compiler.


### spawn(node, fun, opts)

```elixir
@spec spawn(t(), (-&gt; any()), Process.spawn_opts()) :: pid() | {pid(), reference()}
```

Returns the PID of a new process started by the application of `fun`
on `node`.

If `node` does not exist, a useless PID is returned.

For the list of available options, see `:erlang.spawn_opt/3`.

Inlined by the compiler.


### spawn(node, module, fun, args)

```elixir
@spec spawn(t(), module(), atom(), [any()]) :: pid()
```

Returns the PID of a new process started by the application of
`module.function(args)` on `node`.

If `node` does not exist, a useless PID is returned.

For the list of available options, see `:erlang.spawn/4`.

Inlined by the compiler.


### spawn(node, module, fun, args, opts)

```elixir
@spec spawn(t(), module(), atom(), [any()], Process.spawn_opts()) ::
  pid() | {pid(), reference()}
```

Returns the PID of a new process started by the application of
`module.function(args)` on `node`.

If `node` does not exist, a useless PID is returned.

For the list of available options, see `:erlang.spawn_opt/5`.

Inlined by the compiler.


### spawn_link(node, fun)

```elixir
@spec spawn_link(t(), (-&gt; any())) :: pid()
```

Returns the PID of a new linked process started by the application of `fun` on `node`.

A link is created between the calling process and the new process, atomically.
If `node` does not exist, a useless PID is returned (and due to the link, an exit
signal with exit reason `:noconnection` will be received).

Inlined by the compiler.


### spawn_link(node, module, fun, args)

```elixir
@spec spawn_link(t(), module(), atom(), [any()]) :: pid()
```

Returns the PID of a new linked process started by the application of
`module.function(args)` on `node`.

A link is created between the calling process and the new process, atomically.
If `node` does not exist, a useless PID is returned (and due to the link, an exit
signal with exit reason `:noconnection` will be received).

Inlined by the compiler.


### spawn_monitor(node, fun)
*(since 1.14.0)* 
```elixir
@spec spawn_monitor(t(), (-&gt; any())) :: {pid(), reference()}
```

Spawns the given function on a node, monitors it and returns its PID
and monitoring reference.

Inlined by the compiler.


### spawn_monitor(node, module, fun, args)
*(since 1.14.0)* 
```elixir
@spec spawn_monitor(t(), module(), atom(), [any()]) :: {pid(), reference()}
```

Spawns the given module and function passing the given args on a node,
monitors it and returns its PID and monitoring reference.

Inlined by the compiler.


### start(name, type \\ :longnames, tick_time \\ 15000)

```elixir
@spec start(node(), :longnames | :shortnames, non_neg_integer()) ::
  {:ok, pid()} | {:error, term()}
```

Turns a non-distributed node into a distributed node.

This functionality starts the `:net_kernel` and other related
processes.

This function is rarely invoked in practice. Instead, nodes are
named and started via the command line by using the `--sname` and
`--name` flags. If you need to use this function to dynamically
name a node, please make sure the `epmd` operating system process
is running by calling `epmd -daemon`.

Invoking this function when the distribution has already been started,
either via the command line interface or dynamically, will return an
error.

#### Examples

    {:ok, pid} = Node.start(:example, :shortnames, 15000)


### stop()

```elixir
@spec stop() :: :ok | {:error, :not_allowed | :not_found}
```

Turns a distributed node into a non-distributed node.

For other nodes in the network, this is the same as the node going down.
Only possible when the node was started with `Node.start/3`, otherwise
returns `{:error, :not_allowed}`. Returns `{:error, :not_found}` if the
local node is not alive.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
