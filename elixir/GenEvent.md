# GenEvent behaviour
(Elixir v1.18.0-dev)

This behaviour is deprecated. Use Erlang/OTP's :gen_event module instead.
An event manager with event handlers behaviour.

If you are interested in implementing an event manager, please read the
"Alternatives" section below. If you have to implement an event handler to
integrate with an existing system, such as Elixir's Logger, please use
[`:gen_event`](\`:gen_event\`) instead.

## Alternatives

There are a few suitable alternatives to replace GenEvent. Each of them can be
the most beneficial based on the use case.

### Supervisor and GenServers

One alternative to GenEvent is a very minimal solution consisting of using a
supervisor and multiple GenServers started under it. The supervisor acts as
the "event manager" and the children GenServers act as the "event handlers".
This approach has some shortcomings (it provides no back-pressure for example)
but can still replace GenEvent for low-profile usages of it. [This blog post
by José
Valim](https://dashbit.co/blog/replacing-genevent-by-a-supervisor-plus-genserver)
has more detailed information on this approach.

### GenStage

If the use case where you were using GenEvent requires more complex logic,
[GenStage](https://github.com/elixir-lang/gen_stage) provides a great
alternative. GenStage is an external Elixir library maintained by the Elixir
team; it provides a tool to implement systems that exchange events in a
demand-driven way with built-in support for back-pressure. See the [GenStage
documentation](https://hexdocs.pm/gen_stage) for more information.

### `:gen_event`

If your use case requires exactly what GenEvent provided, or you have to
integrate with an existing `:gen_event`-based system, you can still use the
[`:gen_event`](\`:gen_event\`) Erlang module.


## Types

### handler()

```elixir
@type handler() :: atom() | {atom(), term()}
```



### manager()

```elixir
@type manager() :: pid() | name() | {atom(), node()}
```



### name()

```elixir
@type name() :: atom() | {:global, term()} | {:via, module(), term()}
```



### on_start()

```elixir
@type on_start() :: {:ok, pid()} | {:error, {:already_started, pid()}}
```



### options()

```elixir
@type options() :: [{:name, name()}]
```



## Callbacks

### code_change(old_vsn, state, extra)

```elixir
@callback code_change(old_vsn, state :: term(), extra :: term()) ::
  {:ok, new_state :: term()}
when old_vsn: term() | {:down, term()}
```



### handle_call(request, state)

```elixir
@callback handle_call(request :: term(), state :: term()) ::
  {:ok, reply, new_state}
  | {:ok, reply, new_state, :hibernate}
  | {:remove_handler, reply}
when reply: term(), new_state: term()
```



### handle_event(event, state)

```elixir
@callback handle_event(event :: term(), state :: term()) ::
  {:ok, new_state} | {:ok, new_state, :hibernate} | :remove_handler
when new_state: term()
```



### handle_info(msg, state)

```elixir
@callback handle_info(msg :: term(), state :: term()) ::
  {:ok, new_state} | {:ok, new_state, :hibernate} | :remove_handler
when new_state: term()
```



### init(args)

```elixir
@callback init(args :: term()) ::
  {:ok, state} | {:ok, state, :hibernate} | {:error, reason :: term()}
when state: term()
```



### terminate(reason, state)

```elixir
@callback terminate(reason, state :: term()) :: term()
when reason:
       :stop | {:stop, term()} | :remove_handler | {:error, term()} | term()
```





---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
