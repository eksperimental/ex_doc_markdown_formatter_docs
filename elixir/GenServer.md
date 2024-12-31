# GenServer behaviour
(Elixir v1.18.0-dev)

A behaviour module for implementing the server of a client-server relation.

A GenServer is a process like any other Elixir process and it can be used
to keep state, execute code asynchronously and so on. The advantage of using
a generic server process (GenServer) implemented using this module is that it
will have a standard set of interface functions and include functionality for
tracing and error reporting. It will also fit into a supervision tree.

``` mermaid
graph BT
    C(Client #3) ~~~ B(Client #2) ~~~ A(Client #1)
    A & B & C -->|request| GenServer
    GenServer -.->|reply| A & B & C
```

## Example

The GenServer behaviour abstracts the common client-server interaction.
Developers are only required to implement the callbacks and functionality
they are interested in.

Let's start with a code example and then explore the available callbacks.
Imagine we want to implement a service with a GenServer that works
like a stack, allowing us to push and pop elements. We'll customize a
generic GenServer with our own module by implementing three callbacks.

`c:init/1` transforms our initial argument to the initial state for the
GenServer. `c:handle_call/3` fires when the server receives a synchronous
`pop` message, popping an element from the stack and returning it to the
user. `c:handle_cast/2` will fire when the server receives an asynchronous
`push` message, pushing an element onto the stack:

    defmodule Stack do
      use GenServer
    
      # Callbacks
    
      @impl true
      def init(elements) do
        initial_state = String.split(elements, ",", trim: true)
        {:ok, initial_state}
      end
    
      @impl true
      def handle_call(:pop, _from, state) do
        [to_caller | new_state] = state
        {:reply, to_caller, new_state}
      end
    
      @impl true
      def handle_cast({:push, element}, state) do
        new_state = [element | state]
        {:noreply, new_state}
      end
    end

We leave the process machinery of startup, message passing, and the message
loop to the GenServer behaviour and focus only on the stack
implementation. We can now use the GenServer API to interact with
the service by creating a process and sending it messages:

    # Start the server
    {:ok, pid} = GenServer.start_link(Stack, "hello,world")
    
    # This is the client
    GenServer.call(pid, :pop)
    #=> "hello"
    
    GenServer.cast(pid, {:push, "elixir"})
    #=> :ok
    
    GenServer.call(pid, :pop)
    #=> "elixir"

We start our `Stack` by calling `start_link/2`, passing the module
with the server implementation and its initial argument with a
comma-separated list of elements. The GenServer behaviour calls the
`c:init/1` callback to establish the initial GenServer state. From
this point on, the GenServer has control so we interact with it by
sending two types of messages on the client. **call** messages expect
a reply from the server (and are therefore synchronous) while **cast**
messages do not.

Each call to `GenServer.call/3` results in a message
that must be handled by the `c:handle_call/3` callback in the GenServer.
A `cast/2` message must be handled by `c:handle_cast/2`. `GenServer`
supports 8 callbacks, but only `c:init/1` is required.

> #### `use GenServer` {: .info}
> 
> When you `use GenServer`, the `GenServer` module will
> set `@behaviour GenServer` and define a `child_spec/1`
> function, so your module can be used as a child
> in a supervision tree.

## Client / Server APIs

Although in the example above we have used `GenServer.start_link/3` and
friends to directly start and communicate with the server, most of the
time we don't call the `GenServer` functions directly. Instead, we wrap
the calls in new functions representing the public API of the server.
These thin wrappers are called the **client API**.

Here is a better implementation of our Stack module:

    defmodule Stack do
      use GenServer
    
      # Client
    
      def start_link(default) when is_binary(default) do
        GenServer.start_link(__MODULE__, default)
      end
    
      def push(pid, element) do
        GenServer.cast(pid, {:push, element})
      end
    
      def pop(pid) do
        GenServer.call(pid, :pop)
      end
    
      # Server (callbacks)
    
      @impl true
      def init(elements) do
        initial_state = String.split(elements, ",", trim: true)
        {:ok, initial_state}
      end
    
      @impl true
      def handle_call(:pop, _from, state) do
        [to_caller | new_state] = state
        {:reply, to_caller, new_state}
      end
    
      @impl true
      def handle_cast({:push, element}, state) do
        new_state = [element | state]
        {:noreply, new_state}
      end
    end

In practice, it is common to have both server and client functions in
the same module. If the server and/or client implementations are growing
complex, you may want to have them in different modules.

The following diagram summarizes the interactions between client and server.
Both Client and Server are processes and communication happens via messages
(continuous line). The Server \<-\> Module interaction happens when the
GenServer process calls your code (dotted lines):

``` mermaid
sequenceDiagram
    participant C as Client (Process)
    participant S as Server (Process)
    participant M as Module (Code)

    note right of C: Typically started by a supervisor
    C->>+S: GenServer.start_link(module, arg, options)
    S-->>+M: init(arg)
    M-->>-S: {:ok, state} | :ignore | {:error, reason}
    S->>-C: {:ok, pid} | :ignore | {:error, reason}

    note right of C: call is synchronous
    C->>+S: GenServer.call(pid, message)
    S-->>+M: handle_call(message, from, state)
    M-->>-S: {:reply, reply, state} | {:stop, reason, reply, state}
    S->>-C: reply

    note right of C: cast is asynchronous
    C-)S: GenServer.cast(pid, message)
    S-->>+M: handle_cast(message, state)
    M-->>-S: {:noreply, state} | {:stop, reason, state}

    note right of C: send is asynchronous
    C-)S: Kernel.send(pid, message)
    S-->>+M: handle_info(message, state)
    M-->>-S: {:noreply, state} | {:stop, reason, state}
```

## How to supervise

A `GenServer` is most commonly started under a supervision tree.
When we invoke `use GenServer`, it automatically defines a `child_spec/1`
function that allows us to start the `Stack` directly under a supervisor.
To start a default stack of `["hello", "world"]` under a supervisor,
we can do:

    children = [
      {Stack, "hello,world"}
    ]
    
    Supervisor.start_link(children, strategy: :one_for_all)

Note that specifying a module `MyServer` would be the same as specifying
the  tuple `{MyServer, []}`.

`use GenServer` also accepts a list of options which configures the
child specification and therefore how it runs under a supervisor.
The generated `child_spec/1` can be customized with the following options:

- `:id` - the child specification identifier, defaults to the current module
- `:restart` - when the child should be restarted, defaults to `:permanent`
- `:shutdown` - how to shut down the child, either immediately or by giving it time to shut down

For example:

    use GenServer, restart: :transient, shutdown: 10_000

See the "Child specification" section in the `Supervisor` module for more
detailed information. The `@doc` annotation immediately preceding
`use GenServer` will be attached to the generated `child_spec/1` function.

When stopping the GenServer, for example by returning a `{:stop, reason, new_state}`
tuple from a callback, the exit reason is used by the supervisor to determine
whether the GenServer needs to be restarted. See the "Exit reasons and restarts"
section in the `Supervisor` module.

## Name registration

Both `start_link/3` and `start/3` support the `GenServer` to register
a name on start via the `:name` option. Registered names are also
automatically cleaned up on termination. The supported values are:

- an atom - the GenServer is registered locally (to the current node)
  with the given name using `Process.register/2`.

- `{:global, term}` - the GenServer is registered globally with the given
  term using the functions in the [`:global` module](\`:global\`).

- `{:via, module, term}` - the GenServer is registered with the given
  mechanism and name. The `:via` option expects a module that exports
  `register_name/2`, `unregister_name/1`, `whereis_name/1` and `send/2`.
  One such example is the [`:global` module](\`:global\`) which uses these functions
  for keeping the list of names of processes and their associated PIDs
  that are available globally for a network of Elixir nodes. Elixir also
  ships with a local, decentralized and scalable registry called `Registry`
  for locally storing names that are generated dynamically.

For example, we could start and register our `Stack` server locally as follows:

    # Start the server and register it locally with name MyStack
    {:ok, _} = GenServer.start_link(Stack, "hello", name: MyStack)
    
    # Now messages can be sent directly to MyStack
    GenServer.call(MyStack, :pop)
    #=> "hello"

Once the server is started, the remaining functions in this module (`call/3`,
`cast/2`, and friends) will also accept an atom, or any `{:global, ...}` or
`{:via, ...}` tuples. In general, the following formats are supported:

- a PID
- an atom if the server is locally registered
- `{atom, node}` if the server is locally registered at another node
- `{:global, term}` if the server is globally registered
- `{:via, module, name}` if the server is registered through an alternative
  registry

If there is an interest to register dynamic names locally, do not use
atoms, as atoms are never garbage-collected and therefore dynamically
generated atoms won't be garbage-collected. For such cases, you can
set up your own local registry by using the `Registry` module.

## Receiving "regular" messages

The goal of a `GenServer` is to abstract the "receive" loop for developers,
automatically handling system messages, supporting code change, synchronous
calls and more. Therefore, you should never call your own "receive" inside
the GenServer callbacks as doing so will cause the GenServer to misbehave.

Besides the synchronous and asynchronous communication provided by `call/3`
and `cast/2`, "regular" messages sent by functions such as `send/2`,
`Process.send_after/4` and similar, can be handled inside the `c:handle_info/2`
callback.

`c:handle_info/2` can be used in many situations, such as handling monitor
DOWN messages sent by `Process.monitor/1`. Another use case for `c:handle_info/2`
is to perform periodic work, with the help of `Process.send_after/4`:

    defmodule MyApp.Periodically do
      use GenServer
    
      def start_link(_) do
        GenServer.start_link(__MODULE__, %{})
      end
    
      @impl true
      def init(state) do
        # Schedule work to be performed on start
        schedule_work()
    
        {:ok, state}
      end
    
      @impl true
      def handle_info(:work, state) do
        # Do the desired work here
        # ...
    
        # Reschedule once more
        schedule_work()
    
        {:noreply, state}
      end
    
      defp schedule_work do
        # We schedule the work to happen in 2 hours (written in milliseconds).
        # Alternatively, one might write :timer.hours(2)
        Process.send_after(self(), :work, 2 * 60 * 60 * 1000)
      end
    end

## Timeouts

The return value of `c:init/1` or any of the `handle_*` callbacks may include
a timeout value in milliseconds; if not, `:infinity` is assumed.
The timeout can be used to detect a lull in incoming messages.

The `timeout()` value is used as follows:

- If the process has any message already waiting when the `timeout()` value
  is returned, the timeout is ignored and the waiting message is handled as
  usual. This means that even a timeout of `0` milliseconds is not guaranteed
  to execute (if you want to take another action immediately and unconditionally,
  use a `:continue` instruction instead).

- If any message arrives before the specified number of milliseconds
  elapse, the timeout is cleared and that message is handled as usual.

- Otherwise, when the specified number of milliseconds have elapsed with no
  message arriving, `handle_info/2` is called with `:timeout` as the first
  argument.

## When (not) to use a GenServer

So far, we have learned that a `GenServer` can be used as a supervised process
that handles sync and async calls. It can also handle system messages, such as
periodic messages and monitoring events. GenServer processes may also be named.

A GenServer, or a process in general, must be used to model runtime characteristics
of your system. A GenServer must never be used for code organization purposes.

In Elixir, code organization is done by modules and functions, processes are not
necessary. For example, imagine you are implementing a calculator and you decide
to put all the calculator operations behind a GenServer:

    def add(a, b) do
      GenServer.call(__MODULE__, {:add, a, b})
    end
    
    def subtract(a, b) do
      GenServer.call(__MODULE__, {:subtract, a, b})
    end
    
    def handle_call({:add, a, b}, _from, state) do
      {:reply, a + b, state}
    end
    
    def handle_call({:subtract, a, b}, _from, state) do
      {:reply, a - b, state}
    end

This is an anti-pattern not only because it convolutes the calculator logic but
also because you put the calculator logic behind a single process that will
potentially become a bottleneck in your system, especially as the number of
calls grow. Instead just define the functions directly:

    def add(a, b) do
      a + b
    end
    
    def subtract(a, b) do
      a - b
    end

If you don't need a process, then you don't need a process. Use processes only to
model runtime properties, such as mutable state, concurrency and failures, never
for code organization.

## Debugging with the :sys module

GenServers, as [special processes](https://www.erlang.org/doc/design_principles/spec_proc.html),
can be debugged using the [`:sys` module](\`:sys\`).
Through various hooks, this module allows developers to introspect the state of
the process and trace system events that happen during its execution, such as
received messages, sent replies and state changes.

Let's explore the basic functions from the
[`:sys` module](\`:sys\`) used for debugging:

- `:sys.get_state/2` - allows retrieval of the state of the process.
  In the case of a GenServer process, it will be the callback module state,
  as passed into the callback functions as last argument.
- `:sys.get_status/2` - allows retrieval of the status of the process.
  This status includes the process dictionary, if the process is running
  or is suspended, the parent PID, the debugger state, and the state of
  the behaviour module, which includes the callback module state
  (as returned by `:sys.get_state/2`). It's possible to change how this
  status is represented by defining the optional `c:GenServer.format_status/1`
  callback.
- `:sys.trace/3` - prints all the system events to `:stdio`.
- `:sys.statistics/3` - manages collection of process statistics.
- `:sys.no_debug/2` - turns off all debug handlers for the given process.
  It is very important to switch off debugging once we're done. Excessive
  debug handlers or those that should be turned off, but weren't, can
  seriously damage the performance of the system.
- `:sys.suspend/2` - allows to suspend a process so that it only
  replies to system messages but no other messages. A suspended process
  can be reactivated via `:sys.resume/2`.

Let's see how we could use those functions for debugging the stack server
we defined earlier.

    iex> {:ok, pid} = Stack.start_link("")
    iex> :sys.statistics(pid, true) # turn on collecting process statistics
    iex> :sys.trace(pid, true) # turn on event printing
    iex> Stack.push(pid, 1)
    *DBG* <0.122.0> got cast {push,1}
    *DBG* <0.122.0> new state [1]
    :ok
    
    iex> :sys.get_state(pid)
    [1]
    
    iex> Stack.pop(pid)
    *DBG* <0.122.0> got call pop from <0.80.0>
    *DBG* <0.122.0> sent 1 to <0.80.0>, new state []
    1
    
    iex> :sys.statistics(pid, :get)
    {:ok,
     [
       start_time: {{2016, 7, 16}, {12, 29, 41}},
       current_time: {{2016, 7, 16}, {12, 29, 50}},
       reductions: 117,
       messages_in: 2,
       messages_out: 0
     ]}
    
    iex> :sys.no_debug(pid) # turn off all debug handlers
    :ok
    
    iex> :sys.get_status(pid)
    {:status, #PID<0.122.0>, {:module, :gen_server},
     [
       [
         "$initial_call": {Stack, :init, 1},            # process dictionary
         "$ancestors": [#PID<0.80.0>, #PID<0.51.0>]
       ],
       :running,                                        # :running | :suspended
       #PID<0.80.0>,                                    # parent
       [],                                              # debugger state
       [
         header: 'Status for generic server <0.122.0>', # module status
         data: [
           {'Status', :running},
           {'Parent', #PID<0.80.0>},
           {'Logged events', []}
         ],
         data: [{'State', [1]}]
       ]
     ]}

## Learn more

If you wish to find out more about GenServers, the Elixir Getting Started
guide provides a tutorial-like introduction. The documentation and links
in Erlang can also provide extra insight.

- [GenServer - Elixir's Getting Started Guide](genservers.md)
- [`:gen_server` module documentation](\`:gen_server\`)
- [gen\_server Behaviour - OTP Design Principles](https://www.erlang.org/doc/design_principles/gen_server_concepts.html)
- [Clients and Servers - Learn You Some Erlang for Great Good\!](http://learnyousomeerlang.com/clients-and-servers)


## Types

### debug()

```elixir
@type debug() :: [:trace | :log | :statistics | {:log_to_file, Path.t()}]
```

Debug options supported by the `start*` functions


### from()

```elixir
@type from() :: {pid(), tag :: term()}
```

Tuple describing the client of a call request.

`pid` is the PID of the caller and `tag` is a unique term used to identify the
call.


### name()

```elixir
@type name() :: atom() | {:global, term()} | {:via, module(), term()}
```

The GenServer name


### on_start()

```elixir
@type on_start() ::
  {:ok, pid()} | :ignore | {:error, {:already_started, pid()} | term()}
```

Return values of `start*` functions


### option()

```elixir
@type option() ::
  {:debug, debug()}
  | {:name, name()}
  | {:timeout, timeout()}
  | {:spawn_opt, [Process.spawn_opt()]}
  | {:hibernate_after, timeout()}
```

Option values used by the `start*` functions


### options()

```elixir
@type options() :: [option()]
```

Options used by the `start*` functions


### server()

```elixir
@type server() :: pid() | name() | {atom(), node()}
```

The server reference.

This is either a plain PID or a value representing a registered name.
See the "Name registration" section of this document for more information.


## Callbacks

### code_change(old_vsn, state, extra)
*(optional)* 
```elixir
@callback code_change(old_vsn, state :: term(), extra :: term()) ::
  {:ok, new_state :: term()} | {:error, reason :: term()}
when old_vsn: term() | {:down, term()}
```

Invoked to change the state of the `GenServer` when a different version of a
module is loaded (hot code swapping) and the state's term structure should be
changed.

`old_vsn` is the previous version of the module (defined by the `@vsn`
attribute) when upgrading. When downgrading the previous version is wrapped in
a 2-tuple with first element `:down`. `state` is the current state of the
`GenServer` and `extra` is any extra data required to change the state.

Returning `{:ok, new_state}` changes the state to `new_state` and the code
change is successful.

Returning `{:error, reason}` fails the code change with reason `reason` and
the state remains as the previous state.

If `c:code_change/3` raises the code change fails and the loop will continue
with its previous state. Therefore this callback does not usually contain side effects.

This callback is optional.


### format_status(status)
*(since 1.17.0)* *(optional)* 
```elixir
@callback format_status(status :: :gen_server.format_status()) ::
  new_status :: :gen_server.format_status()
```

This function is called by a `GenServer` process in the following situations:

- [`:sys.get_status/1,2`](\`:sys.get_status/1\`) is invoked to get the `GenServer` status.
- The `GenServer` process terminates abnormally and logs an error.

This callback is used to limit the status of the process returned by
[`:sys.get_status/1,2`](\`:sys.get_status/1\`) or sent to logger.

The callback gets a map `status` describing the current status and shall return
a map `new_status` with the same keys, but it may transform some values.

Two possible use cases for this callback is to remove sensitive information
from the state to prevent it from being printed in log files, or to compact
large irrelevant status items that would only clutter the logs.

#### Example

    @impl GenServer
    def format_status(status) do
      Map.new(status, fn
        {:state, state} -> {:state, Map.delete(state, :private_key)}
        {:message, {:password, _}} -> {:message, {:password, "redacted"}}
        key_value -> key_value
      end)
    end


### format_status(reason, pdict_and_state)
*(optional)* 
```elixir
@callback format_status(reason, pdict_and_state :: list()) :: term()
when reason: :normal | :terminate
```
This callback is deprecated. Use format_status/1 callback instead.


### handle_call(request, from, state)
*(optional)* 
```elixir
@callback handle_call(request :: term(), from(), state :: term()) ::
  {:reply, reply, new_state}
  | {:reply, reply, new_state,
     timeout() | :hibernate | {:continue, continue_arg :: term()}}
  | {:noreply, new_state}
  | {:noreply, new_state,
     timeout() | :hibernate | {:continue, continue_arg :: term()}}
  | {:stop, reason, reply, new_state}
  | {:stop, reason, new_state}
when reply: term(), new_state: term(), reason: term()
```

Invoked to handle synchronous `call/3` messages. `call/3` will block until a
reply is received (unless the call times out or nodes are disconnected).

`request` is the request message sent by a `call/3`, `from` is a 2-tuple
containing the caller's PID and a term that uniquely identifies the call, and
`state` is the current state of the `GenServer`.

Returning `{:reply, reply, new_state}` sends the response `reply` to the
caller and continues the loop with new state `new_state`.

Returning `{:reply, reply, new_state, timeout}` is similar to
`{:reply, reply, new_state}` except that it also sets a timeout.
See the "Timeouts" section in the module documentation for more information.

Returning `{:reply, reply, new_state, :hibernate}` is similar to
`{:reply, reply, new_state}` except the process is hibernated and will
continue the loop once a message is in its message queue. However, if a message is
already in the message queue, the process will continue the loop immediately.
Hibernating a `GenServer` causes garbage collection and leaves a continuous
heap that minimises the memory used by the process.

Hibernating should not be used aggressively as too much time could be spent
garbage collecting, which would delay the processing of incoming messages.
Normally it should only be used when you are not expecting new messages to
immediately arrive and minimising the memory of the process is shown to be
beneficial.

Returning `{:reply, reply, new_state, {:continue, continue_arg}}` is similar to
`{:reply, reply, new_state}` except that `c:handle_continue/2` will be invoked
immediately after with `continue_arg` as the first argument and
`state` as the second one.

Returning `{:noreply, new_state}` does not send a response to the caller and
continues the loop with new state `new_state`. The response must be sent with
`reply/2`.

There are three main use cases for not replying using the return value:

- To reply before returning from the callback because the response is known
  before calling a slow function.
- To reply after returning from the callback because the response is not yet
  available.
- To reply from another process, such as a task.

When replying from another process the `GenServer` should exit if the other
process exits without replying as the caller will be blocking awaiting a
reply.

Returning `{:noreply, new_state, timeout | :hibernate | {:continue, continue_arg}}`
is similar to `{:noreply, new_state}` except a timeout, hibernation or continue
occurs as with a `:reply` tuple.

Returning `{:stop, reason, reply, new_state}` stops the loop and `c:terminate/2`
is called with reason `reason` and state `new_state`. Then, the `reply` is sent
as the response to call and the process exits with reason `reason`.

Returning `{:stop, reason, new_state}` is similar to
`{:stop, reason, reply, new_state}` except a reply is not sent.

This callback is optional. If one is not implemented, the server will fail
if a call is performed against it.


### handle_cast(request, state)
*(optional)* 
```elixir
@callback handle_cast(request :: term(), state :: term()) ::
  {:noreply, new_state}
  | {:noreply, new_state,
     timeout() | :hibernate | {:continue, continue_arg :: term()}}
  | {:stop, reason :: term(), new_state}
when new_state: term()
```

Invoked to handle asynchronous `cast/2` messages.

`request` is the request message sent by a `cast/2` and `state` is the current
state of the `GenServer`.

Returning `{:noreply, new_state}` continues the loop with new state `new_state`.

Returning `{:noreply, new_state, timeout}` is similar to `{:noreply, new_state}`
except that it also sets a timeout. See the "Timeouts" section in the module
documentation for more information.

Returning `{:noreply, new_state, :hibernate}` is similar to
`{:noreply, new_state}` except the process is hibernated before continuing the
loop. See `c:handle_call/3` for more information.

Returning `{:noreply, new_state, {:continue, continue_arg}}` is similar to
`{:noreply, new_state}` except `c:handle_continue/2` will be invoked
immediately after with `continue_arg` as the first argument and
`state` as the second one.

Returning `{:stop, reason, new_state}` stops the loop and `c:terminate/2` is
called with the reason `reason` and state `new_state`. The process exits with
reason `reason`.

This callback is optional. If one is not implemented, the server will fail
if a cast is performed against it.


### handle_continue(continue_arg, state)
*(optional)* 
```elixir
@callback handle_continue(continue_arg, state :: term()) ::
  {:noreply, new_state}
  | {:noreply, new_state, timeout() | :hibernate | {:continue, continue_arg}}
  | {:stop, reason :: term(), new_state}
when new_state: term(), continue_arg: term()
```

Invoked to handle continue instructions.

It is useful for performing work after initialization or for splitting the work
in a callback in multiple steps, updating the process state along the way.

Return values are the same as `c:handle_cast/2`.

This callback is optional. If one is not implemented, the server will fail
if a continue instruction is used.


### handle_info(msg, state)
*(optional)* 
```elixir
@callback handle_info(msg :: :timeout | term(), state :: term()) ::
  {:noreply, new_state}
  | {:noreply, new_state,
     timeout() | :hibernate | {:continue, continue_arg :: term()}}
  | {:stop, reason :: term(), new_state}
when new_state: term()
```

Invoked to handle all other messages.

`msg` is the message and `state` is the current state of the `GenServer`. When
a timeout occurs the message is `:timeout`.

Return values are the same as `c:handle_cast/2`.

This callback is optional. If one is not implemented, the received message
will be logged.


### init(init_arg)

```elixir
@callback init(init_arg :: term()) ::
  {:ok, state}
  | {:ok, state, timeout() | :hibernate | {:continue, continue_arg :: term()}}
  | :ignore
  | {:stop, reason :: term()}
when state: term()
```

Invoked when the server is started. `start_link/3` or `start/3` will
block until it returns.

`init_arg` is the argument term (second argument) passed to `start_link/3`.

Returning `{:ok, state}` will cause `start_link/3` to return
`{:ok, pid}` and the process to enter its loop.

Returning `{:ok, state, timeout}` is similar to `{:ok, state}`,
except that it also sets a timeout. See the "Timeouts" section
in the module documentation for more information.

Returning `{:ok, state, :hibernate}` is similar to `{:ok, state}`
except the process is hibernated before entering the loop. See
`c:handle_call/3` for more information on hibernation.

Returning `{:ok, state, {:continue, continue_arg}}` is similar to
`{:ok, state}` except that immediately after entering the loop,
the `c:handle_continue/2` callback will be invoked with `continue_arg`
as the first argument and `state` as the second one.

Returning `:ignore` will cause `start_link/3` to return `:ignore` and
the process will exit normally without entering the loop or calling
`c:terminate/2`. If used when part of a supervision tree the parent
supervisor will not fail to start nor immediately try to restart the
`GenServer`. The remainder of the supervision tree will be started
and so the `GenServer` should not be required by other processes.
It can be started later with `Supervisor.restart_child/2` as the child
specification is saved in the parent supervisor. The main use cases for
this are:

- The `GenServer` is disabled by configuration but might be enabled later.
- An error occurred and it will be handled by a different mechanism than the
  `Supervisor`. Likely this approach involves calling `Supervisor.restart_child/2`
  after a delay to attempt a restart.

Returning `{:stop, reason}` will cause `start_link/3` to return
`{:error, reason}` and the process to exit with reason `reason` without
entering the loop or calling `c:terminate/2`.


### terminate(reason, state)
*(optional)* 
```elixir
@callback terminate(reason, state :: term()) :: term()
when reason: :normal | :shutdown | {:shutdown, term()} | term()
```

Invoked when the server is about to exit. It should do any cleanup required.

`reason` is exit reason and `state` is the current state of the `GenServer`.
The return value is ignored.

`c:terminate/2` is useful for cleanup that requires access to the
`GenServer`'s state. However, it is **not guaranteed** that `c:terminate/2`
is called when a `GenServer` exits. Therefore, important cleanup should be
done using process links and/or monitors. A monitoring process will receive the
same exit `reason` that would be passed to `c:terminate/2`.

`c:terminate/2` is called if:

- the `GenServer` traps exits (using `Process.flag/2`) *and* the parent
  process (the one which called `start_link/1`) sends an exit signal

- a callback (except `c:init/1`) does one of the following:
  
  - returns a `:stop` tuple
  
  - raises (via `raise/2`) or exits (via `exit/1`)
  
  - returns an invalid value

If part of a supervision tree, a `GenServer` will receive an exit signal from
its parent process (its supervisor) when the tree is shutting down. The exit
signal is based on the shutdown strategy in the child's specification, where
this value can be:

- `:brutal_kill`: the `GenServer` is killed and so `c:terminate/2` is not called.

- a timeout value, where the supervisor will send the exit signal `:shutdown` and
  the `GenServer` will have the duration of the timeout to terminate.
  If after duration of this timeout the process is still alive, it will be killed
  immediately.

For a more in-depth explanation, please read the "Shutdown values (:shutdown)"
section in the `Supervisor` module.

If the `GenServer` receives an exit signal (that is not `:normal`) from any
process when it is not trapping exits it will exit abruptly with the same
reason and so not call `c:terminate/2`. Note that a process does *NOT* trap
exits by default and an exit signal is sent when a linked process exits or its
node is disconnected.

`c:terminate/2` is only called after the `GenServer` finishes processing all
messages which arrived in its mailbox prior to the exit signal. If it
receives a `:kill` signal before it finishes processing those,
`c:terminate/2` will not be called. If `c:terminate/2` is called, any
messages received after the exit signal will still be in the mailbox.

There is no cleanup needed when the `GenServer` controls a `port` (for example,
`:gen_tcp.socket`) or `t:File.io_device/0`, because these will be closed on
receiving a `GenServer`'s exit signal and do not need to be closed manually
in `c:terminate/2`.

If `reason` is neither `:normal`, `:shutdown`, nor `{:shutdown, term}` an error is
logged.

This callback is optional.


## Functions

### abcast(nodes \\ [node() | Node.list()], name, request)

```elixir
@spec abcast([node()], name :: atom(), term()) :: :abcast
```

Casts all servers locally registered as `name` at the specified nodes.

This function returns immediately and ignores nodes that do not exist, or where the
server name does not exist.

See `multi_call/4` for more information.


### call(server, request, timeout \\ 5000)

```elixir
@spec call(server(), term(), timeout()) :: term()
```

Makes a synchronous call to the `server` and waits for its reply.

The client sends the given `request` to the server and waits until a reply
arrives or a timeout occurs. `c:handle_call/3` will be called on the server
to handle the request.

`server` can be any of the values described in the "Name registration"
section of the documentation for this module.

#### Timeouts

`timeout` is an integer greater than zero which specifies how many
milliseconds to wait for a reply, or the atom `:infinity` to wait
indefinitely. The default value is `5000`. If no reply is received within
the specified time, the function call fails and the caller exits. If the
caller catches the failure and continues running, and the server is just late
with the reply, it may arrive at any time later into the caller's message
queue. The caller must in this case be prepared for this and discard any such
garbage messages that are two-element tuples with a reference as the first
element.


### cast(server, request)

```elixir
@spec cast(server(), term()) :: :ok
```

Casts a request to the `server` without waiting for a response.

This function always returns `:ok` regardless of whether
the destination `server` (or node) exists. Therefore it
is unknown whether the destination `server` successfully
handled the request.

`server` can be any of the values described in the "Name registration"
section of the documentation for this module.


### multi_call(nodes \\ [node() | Node.list()], name, request, timeout \\ :infinity)

```elixir
@spec multi_call([node()], name :: atom(), term(), timeout()) ::
  {replies :: [{node(), term()}], bad_nodes :: [node()]}
```

Calls all servers locally registered as `name` at the specified `nodes`.

First, the `request` is sent to every node in `nodes`; then, the caller waits
for the replies. This function returns a two-element tuple `{replies, bad_nodes}` where:

- `replies` - is a list of `{node, reply}` tuples where `node` is the node
  that replied and `reply` is its reply
- `bad_nodes` - is a list of nodes that either did not exist or where a
  server with the given `name` did not exist or did not reply

`nodes` is a list of node names to which the request is sent. The default
value is the list of all known nodes (including this node).

#### Examples

Assuming the `Stack` GenServer mentioned in the docs for the `GenServer`
module is registered as `Stack` in the `:"foo@my-machine"` and
`:"bar@my-machine"` nodes:

    GenServer.multi_call(Stack, :pop)
    #=> {[{:"foo@my-machine", :hello}, {:"bar@my-machine", :world}], []}


### reply(client, reply)

```elixir
@spec reply(from(), term()) :: :ok
```

Replies to a client.

This function can be used to explicitly send a reply to a client that called
`call/3` or `multi_call/4` when the reply cannot be specified in the return
value of `c:handle_call/3`.

`client` must be the `from` argument (the second argument) accepted by
`c:handle_call/3` callbacks. `reply` is an arbitrary term which will be given
back to the client as the return value of the call.

Note that `reply/2` can be called from any process, not just the GenServer
that originally received the call (as long as that GenServer communicated the
`from` argument somehow).

This function always returns `:ok`.

#### Examples

    def handle_call(:reply_in_one_second, from, state) do
      Process.send_after(self(), {:reply, from}, 1_000)
      {:noreply, state}
    end
    
    def handle_info({:reply, from}, state) do
      GenServer.reply(from, :one_second_has_passed)
      {:noreply, state}
    end


### start(module, init_arg, options \\ [])

```elixir
@spec start(module(), term(), options()) :: on_start()
```

Starts a `GenServer` process without links (outside of a supervision tree).

See `start_link/3` for more information.


### start_link(module, init_arg, options \\ [])

```elixir
@spec start_link(module(), term(), options()) :: on_start()
```

Starts a `GenServer` process linked to the current process.

This is often used to start the `GenServer` as part of a supervision tree.

Once the server is started, the `c:init/1` function of the given `module` is
called with `init_arg` as its argument to initialize the server. To ensure a
synchronized start-up procedure, this function does not return until `c:init/1`
has returned.

Note that a `GenServer` started with `start_link/3` is linked to the
parent process and will exit in case of crashes from the parent. The GenServer
will also exit due to the `:normal` reasons in case it is configured to trap
exits in the `c:init/1` callback.

#### Options

- `:name` - used for name registration as described in the "Name
  registration" section in the documentation for `GenServer`

- `:timeout` - if present, the server is allowed to spend the given number of
  milliseconds initializing or it will be terminated and the start function
  will return `{:error, :timeout}`

- `:debug` - if present, the corresponding function in the [`:sys` module](\`:sys\`) is invoked

- `:spawn_opt` - if present, its value is passed as options to the
  underlying process as in `Process.spawn/4`

- `:hibernate_after` - if present, the GenServer process awaits any message for
  the given number of milliseconds and if no message is received, the process goes
  into hibernation automatically (by calling `:proc_lib.hibernate/3`).

#### Return values

If the server is successfully created and initialized, this function returns
`{:ok, pid}`, where `pid` is the PID of the server. If a process with the
specified server name already exists, this function returns
`{:error, {:already_started, pid}}` with the PID of that process.

If the `c:init/1` callback fails with `reason`, this function returns
`{:error, reason}`. Otherwise, if it returns `{:stop, reason}`
or `:ignore`, the process is terminated and this function returns
`{:error, reason}` or `:ignore`, respectively.


### stop(server, reason \\ :normal, timeout \\ :infinity)

```elixir
@spec stop(server(), reason :: term(), timeout()) :: :ok
```

Synchronously stops the server with the given `reason`.

The `c:terminate/2` callback of the given `server` will be invoked before
exiting. This function returns `:ok` if the server terminates with the
given reason; if it terminates with another reason, the call exits.

This function keeps OTP semantics regarding error reporting.
If the reason is any other than `:normal`, `:shutdown` or
`{:shutdown, _}`, an error report is logged.


### whereis(server)

```elixir
@spec whereis(server()) :: pid() | {atom(), node()} | nil
```

Returns the `pid` or `{name, node}` of a GenServer process, `nil` otherwise.

To be precise, `nil` is returned whenever a `pid` or `{name, node}` cannot
be returned. Note there is no guarantee the returned `pid` or `{name, node}`
is alive, as a process could terminate immediately after it is looked up.

#### Examples

For example, to lookup a server process, monitor it and send a cast to it:

    process = GenServer.whereis(server)
    monitor = Process.monitor(process)
    GenServer.cast(process, :hello)




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").