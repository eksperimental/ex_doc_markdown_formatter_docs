# System 
(Elixir v1.18.0-dev)

The `System` module provides functions that interact directly
with the VM or the host system.

## Time

The `System` module also provides functions that work with time,
returning different times kept by the system with support for
different time units.

One of the complexities in relying on system times is that they
may be adjusted. For example, when you enter and leave daylight
saving time, the system clock will be adjusted, often adding
or removing one hour. We call such changes "time warps". In
order to understand how such changes may be harmful, imagine
the following code:

    ## DO NOT DO THIS
    prev = System.os_time()
    # ... execute some code ...
    next = System.os_time()
    diff = next - prev

If, while the code is executing, the system clock changes,
some code that executed in 1 second may be reported as taking
over 1 hour\! To address such concerns, the VM provides a
monotonic time via `System.monotonic_time/0` which never
decreases and does not leap:

    ## DO THIS
    prev = System.monotonic_time()
    # ... execute some code ...
    next = System.monotonic_time()
    diff = next - prev

Generally speaking, the VM provides three time measurements:

- `os_time/0` - the time reported by the operating system (OS). This time may be
  adjusted forwards or backwards in time with no limitation;

- `system_time/0` - the VM view of the `os_time/0`. The system time and operating
  system time may not match in case of time warps although the VM works towards
  aligning them. This time is not monotonic (i.e., it may decrease)
  as its behavior is configured [by the VM time warp
  mode](https://www.erlang.org/doc/apps/erts/time_correction.html#Time_Warp_Modes);

- `monotonic_time/0` - a monotonically increasing time provided
  by the Erlang VM. This is not strictly monotonically increasing. Multiple
  sequential calls of the function may return the same value.

The time functions in this module work in the `:native` unit
(unless specified otherwise), which is operating system dependent. Most of
the time, all calculations are done in the `:native` unit, to
avoid loss of precision, with `convert_time_unit/3` being
invoked at the end to convert to a specific time unit like
`:millisecond` or `:microsecond`. See the `t:time_unit/0` type for
more information.

For a more complete rundown on the VM support for different
times, see the [chapter on time and time
correction](https://www.erlang.org/doc/apps/erts/time_correction.html)
in the Erlang docs.


## Types

### signal()

```elixir
@type signal() ::
  :sigabrt
  | :sigalrm
  | :sigchld
  | :sighup
  | :sigquit
  | :sigstop
  | :sigterm
  | :sigtstp
  | :sigusr1
  | :sigusr2
```



### time_unit()

```elixir
@type time_unit() ::
  :second | :millisecond | :microsecond | :nanosecond | pos_integer()
```

The time unit to be passed to functions like `monotonic_time/1` and others.

The `:second`, `:millisecond`, `:microsecond` and `:nanosecond` time
units controls the return value of the functions that accept a time unit.

A time unit can also be a strictly positive integer. In this case, it
represents the "parts per second": the time will be returned in `1 / parts_per_second` seconds. For example, using the `:millisecond` time unit
is equivalent to using `1000` as the time unit (as the time will be returned
in 1/1000 seconds - milliseconds).


## Functions

### argv()

```elixir
@spec argv() :: [String.t()]
```

Lists command line arguments.

Returns the list of command line arguments passed to the program.


### argv(args)

```elixir
@spec argv([String.t()]) :: :ok
```

Modifies command line arguments.

Changes the list of command line arguments. Use it with caution,
as it destroys any previous argv information.


### at_exit(fun)

```elixir
@spec at_exit((non_neg_integer() -&gt; any())) :: :ok
```

Registers a program exit handler function.

Registers a function that will be invoked at the end of an Elixir script.
A script is typically started via the command line via the `elixir` and
`mix` executables.

The handler always executes in a different process from the one it was
registered in. As a consequence, any resources managed by the calling process
(ETS tables, open files, and others) won't be available by the time the handler
function is invoked.

The function must receive the exit status code as an argument.

If the VM terminates programmatically, via `System.stop/1`, `System.halt/1`,
or exit signals, the `at_exit/1` callbacks are not guaranteed to be executed.


### build_info()

```elixir
@spec build_info() :: %{
  build: String.t(),
  date: String.t(),
  revision: String.t(),
  version: String.t(),
  otp_release: String.t()
}
```

Elixir build information.

Returns a map with the Elixir version, the Erlang/OTP release it was compiled
with, a short Git revision hash and the date and time it was built.

Every value in the map is a string, and these are:

- `:build` - the Elixir version, short Git revision hash and
  Erlang/OTP release it was compiled with
- `:date` - a string representation of the ISO8601 date and time it was built
- `:otp_release` - OTP release it was compiled with
- `:revision` - short Git revision hash. If Git was not available at building
  time, it is set to `""`
- `:version` - the Elixir version

One should not rely on the specific formats returned by each of those fields.
Instead one should use specialized functions, such as `version/0` to retrieve
the Elixir version and `otp_release/0` to retrieve the Erlang/OTP release.

#### Examples

    iex> System.build_info()
    %{
      build: "1.9.0-dev (772a00a0c) (compiled with Erlang/OTP 21)",
      date: "2018-12-24T01:09:21Z",
      otp_release: "21",
      revision: "772a00a0c",
      version: "1.9.0-dev"
    }


### cmd(command, args, opts \\ [])

```elixir
@spec cmd(binary(), [binary()], keyword()) ::
  {Collectable.t(), exit_status :: non_neg_integer()}
```

Executes the given `command` with `args`.

`command` is expected to be an executable available in PATH
unless an absolute path is given.

`args` must be a list of binaries which the executable will receive
as its arguments as is. This means that:

- environment variables will not be interpolated
- wildcard expansion will not happen (unless `Path.wildcard/2` is used
  explicitly)
- arguments do not need to be escaped or quoted for shell safety

This function returns a tuple containing the collected result
and the command exit status.

Internally, this function uses a `Port` for interacting with the
outside world. However, if you plan to run a long-running program,
ports guarantee stdin/stdout devices will be closed but it does not
automatically terminate the program. The documentation for the
`Port` module describes this problem and possible solutions under
the "Zombie processes" section.

> #### Windows argument splitting and untrusted arguments {: .warning}
> 
> On Unix systems, arguments are passed to a new operating system
> process as an array of strings but on Windows it is up to the child
> process to parse them and some Windows programs may apply their own
> rules, which are inconsistent with the standard C runtime `argv` parsing
> 
> This is particularly troublesome when invoking `.bat` or `.com` files
> as these run implicitly through `cmd.exe`, whose argument parsing is
> vulnerable to malicious input and can be used to run arbitrary shell
> commands.
> 
> Therefore, if you are running on Windows and you execute batch
> files or `.com` applications, you must not pass untrusted input as
> arguments to the program. You may avoid accidentally executing them
> by explicitly passing the extension of the program you want to run,
> such as `.exe`, and double check the program is indeed not a batch
> file or `.com` application.

#### Examples

    iex> System.cmd("echo", ["hello"])
    {"hello\n", 0}
    
    iex> System.cmd("echo", ["hello"], env: [{"MIX_ENV", "test"}])
    {"hello\n", 0}

If you want to stream the output to Standard IO as it arrives:

    iex> System.cmd("echo", ["hello"], into: IO.stream())
    hello
    {%IO.Stream{}, 0}

If you want to read lines:

    iex> System.cmd("echo", ["hello\nworld"], into: [], lines: 1024)
    {["hello", "world"], 0}

#### Options

- `:into` - injects the result into the given collectable, defaults to `""`

- `:lines` - (since v1.15.0) reads the output by lines instead of in bytes. It expects a
  number of maximum bytes to buffer internally (1024 is a reasonable default).
  The collectable will be called with each finished line (regardless of buffer
  size) and without the EOL character

- `:cd` - the directory to run the command in

- `:env` - an enumerable of tuples containing environment key-value as
  binary. The child process inherits all environment variables from its
  parent process, the Elixir application, except those overwritten or
  cleared using this option. Specify a value of `nil` to clear (unset) an
  environment variable, which is useful for preventing credentials passed
  to the application from leaking into child processes

- `:arg0` - sets the command arg0

- `:stderr_to_stdout` - redirects stderr to stdout when `true`, no effect
  if `use_stdio` is `false`.

- `:use_stdio` - `true` by default, setting it to false allows direct
  interaction with the terminal from the callee

- `:parallelism` - when `true`, the VM will schedule port tasks to improve
  parallelism in the system. If set to `false`, the VM will try to perform
  commands immediately, improving latency at the expense of parallelism.
  The default is `false`, and can be set on system startup by passing the
  [`+spp`](https://www.erlang.org/doc/man/erl.html#+spp) flag to `--erl`.
  Use `:erlang.system_info(:port_parallelism)` to check if enabled.

#### Error reasons

If invalid arguments are given, `ArgumentError` is raised by
`System.cmd/3`. `System.cmd/3` also expects a strict set of
options and will raise if unknown or invalid options are given.

Furthermore, `System.cmd/3` may fail with one of the POSIX reasons
detailed below:

- `:system_limit` - all available ports in the Erlang emulator are in use

- `:enomem` - there was not enough memory to create the port

- `:eagain` - there are no more available operating system processes

- `:enametoolong` - the external command given was too long

- `:emfile` - there are no more available file descriptors
  (for the operating system process that the Erlang emulator runs in)

- `:enfile` - the file table is full (for the entire operating system)

- `:eacces` - the command does not point to an executable file

- `:enoent` - the command does not point to an existing file

#### Shell commands

If you desire to execute a trusted command inside a shell, with pipes,
redirecting and so on, please check `shell/2`.


### compiled_endianness()

```elixir
@spec compiled_endianness() :: :little | :big
```

Returns the endianness the system was compiled with.


### convert_time_unit(time, from_unit, to_unit)

```elixir
@spec convert_time_unit(integer(), time_unit() | :native, time_unit() | :native) ::
  integer()
```

Converts `time` from time unit `from_unit` to time unit `to_unit`.

The result is rounded via the floor function.

`convert_time_unit/3` accepts an additional time unit (other than the
ones in the `t:time_unit/0` type) called `:native`. `:native` is the time
unit used by the Erlang runtime system. It's determined when the runtime
starts and stays the same until the runtime is stopped, but could differ
the next time the runtime is started on the same machine. For this reason,
you should use this function to convert `:native` time units to a predictable
unit before you display them to humans.

To determine how many seconds the `:native` unit represents in your current
runtime, you can call this function to convert 1 second to the `:native`
time unit: `System.convert_time_unit(1, :second, :native)`.


### cwd()

```elixir
@spec cwd() :: String.t() | nil
```
This function is deprecated. Use File.cwd/0 instead.
Current working directory.

Returns the current working directory or `nil` if one
is not available.


### cwd!()

```elixir
@spec cwd!() :: String.t()
```
This function is deprecated. Use File.cwd!/0 instead.
Current working directory, exception on error.

Returns the current working directory or raises `RuntimeError`.


### delete_env(varname)

```elixir
@spec delete_env(String.t()) :: :ok
```

Deletes an environment variable.

Removes the variable `varname` from the environment.


### endianness()

```elixir
@spec endianness() :: :little | :big
```

Returns the endianness.


### fetch_env(varname)
*(since 1.9.0)* 
```elixir
@spec fetch_env(String.t()) :: {:ok, String.t()} | :error
```

Returns the value of the given environment variable or `:error` if not found.

If the environment variable `varname` is set, then `{:ok, value}` is returned
where `value` is a string. If `varname` is not set, `:error` is returned.

#### Examples

    iex> System.fetch_env("PORT")
    {:ok, "4000"}
    
    iex> System.fetch_env("NOT_SET")
    :error


### fetch_env!(varname)
*(since 1.9.0)* 
```elixir
@spec fetch_env!(String.t()) :: String.t()
```

Returns the value of the given environment variable or raises if not found.

Same as `get_env/1` but raises instead of returning `nil` when the variable is
not set.

#### Examples

    iex> System.fetch_env!("PORT")
    "4000"
    
    iex> System.fetch_env!("NOT_SET")
    ** (System.EnvError) could not fetch environment variable "NOT_SET" because it is not set


### find_executable(program)

```elixir
@spec find_executable(binary()) :: binary() | nil
```

Locates an executable on the system.

This function looks up an executable program given
its name using the environment variable PATH on Windows and Unix-like
operating systems. It also considers the proper executable
extension for each operating system, so for Windows it will try to
lookup files with `.com`, `.cmd` or similar extensions.


### get_env()

```elixir
@spec get_env() :: %{optional(String.t()) =&gt; String.t()}
```

Returns all system environment variables.

The returned value is a map containing name-value pairs.
Variable names and their values are strings.


### get_env(varname, default \\ nil)
*(since 1.9.0)* 
```elixir
@spec get_env(String.t(), String.t()) :: String.t()
@spec get_env(String.t(), nil) :: String.t() | nil
```

Returns the value of the given environment variable.

The returned value of the environment variable
`varname` is a string. If the environment variable
is not set, returns the string specified in `default` or
`nil` if none is specified.

#### Examples

    iex> System.get_env("PORT")
    "4000"
    
    iex> System.get_env("NOT_SET")
    nil
    
    iex> System.get_env("NOT_SET", "4001")
    "4001"


### get_pid()

```elixir
@spec get_pid() :: binary()
```
This function is deprecated. Use System.pid/0 instead.
Erlang VM process identifier.

Returns the process identifier of the current Erlang emulator
in the format most commonly used by the operating system environment.

For more information, see `:os.getpid/0`.


### halt(status \\ 0)

```elixir
@spec halt(non_neg_integer() | binary() | :abort) :: no_return()
```

Immediately halts the Erlang runtime system.

Terminates the Erlang runtime system without properly shutting down
applications and ports. Please see `stop/1` for a careful shutdown of the
system.

`status` must be a non-negative integer, the atom `:abort` or a binary.

- If an integer, the runtime system exits with the integer value which
  is returned to the operating system.

- If `:abort`, the runtime system aborts producing a core dump, if that is
  enabled in the operating system.

- If a string, an Erlang crash dump is produced with status as slogan,
  and then the runtime system exits with status code 1.

Note that on many platforms, only the status codes 0-255 are supported
by the operating system.

For more information, see `:erlang.halt/1`.

#### Examples

    System.halt(0)
    System.halt(1)
    System.halt(:abort)


### monotonic_time()

```elixir
@spec monotonic_time() :: integer()
```

Returns the current monotonic time in the `:native` time unit.

This time is monotonically increasing and starts in an unspecified
point in time. This is not strictly monotonically increasing. Multiple
sequential calls of the function may return the same value.

Inlined by the compiler.


### monotonic_time(unit)

```elixir
@spec monotonic_time(time_unit() | :native) :: integer()
```

Returns the current monotonic time in the given time unit.

This time is monotonically increasing and starts in an unspecified
point in time.


### no_halt()
*(since 1.9.0)* 
```elixir
@spec no_halt() :: boolean()
```

Checks if the system will halt or not at the end of ARGV processing.


### no_halt(boolean)
*(since 1.9.0)* 
```elixir
@spec no_halt(boolean()) :: :ok
```

Marks if the system should halt or not at the end of ARGV processing.


### os_time()
*(since 1.3.0)* 
```elixir
@spec os_time() :: integer()
```

Returns the current operating system (OS) time.

The result is returned in the `:native` time unit.

This time may be adjusted forwards or backwards in time
with no limitation and is not monotonic.

Inlined by the compiler.


### os_time(unit)
*(since 1.3.0)* 
```elixir
@spec os_time(time_unit() | :native) :: integer()
```

Returns the current operating system (OS) time in the given time `unit`.

This time may be adjusted forwards or backwards in time
with no limitation and is not monotonic.


### otp_release()
*(since 1.3.0)* 
```elixir
@spec otp_release() :: String.t()
```

Returns the Erlang/OTP release number.


### pid()
*(since 1.9.0)* 
```elixir
@spec pid() :: String.t()
```

Returns the operating system PID for the current Erlang runtime system instance.

Returns a string containing the (usually) numerical identifier for a process.
On Unix-like operating systems, this is typically the return value of the `getpid()` system call.
On Windows, the process ID as returned by the `GetCurrentProcessId()` system
call is used.

#### Examples

    System.pid()


### put_env(enum)

```elixir
@spec put_env(Enumerable.t()) :: :ok
```

Sets multiple environment variables.

Sets a new value for each environment variable corresponding
to each `{key, value}` pair in `enum`. Keys and non-nil values
are automatically converted to charlists. `nil` values erase
the given keys.

Overall, this is a convenience wrapper around `put_env/2` and
`delete_env/2` with support for different key and value formats.


### put_env(varname, value)

```elixir
@spec put_env(binary(), binary()) :: :ok
```

Sets an environment variable value.

Sets a new `value` for the environment variable `varname`.


### restart()
*(since 1.9.0)* 
```elixir
@spec restart() :: :ok
```

Restarts all applications in the Erlang runtime system.

All applications are taken down smoothly, all code is unloaded, and all ports
are closed before the system starts all applications once again.

#### Examples

    System.restart()


### schedulers()
*(since 1.3.0)* 
```elixir
@spec schedulers() :: pos_integer()
```

Returns the number of schedulers in the VM.


### schedulers_online()
*(since 1.3.0)* 
```elixir
@spec schedulers_online() :: pos_integer()
```

Returns the number of schedulers online in the VM.


### shell(command, opts \\ [])
*(since 1.12.0)* 
```elixir
@spec shell(
  binary(),
  keyword()
) :: {Collectable.t(), exit_status :: non_neg_integer()}
```

Executes the given `command` in the OS shell.

It uses `sh` for Unix-like systems and `cmd` for Windows.

> #### Watch out {: .warning}
> 
> Use this function with care. In particular, **never
> pass untrusted user input to this function**, as the user would be
> able to perform "command injection attacks" by executing any code
> directly on the machine. Generally speaking, prefer to use `cmd/3`
> over this function.

#### Examples

    iex> System.shell("echo hello")
    {"hello\n", 0}

If you want to stream the output to Standard IO as it arrives:

    iex> System.shell("echo hello", into: IO.stream())
    hello
    {%IO.Stream{}, 0}

#### Options

It accepts the same options as `cmd/3` (except for `arg0`).
It also accepts the following exclusive options:

- `:close_stdin` (since v1.14.1) - if the stdin should be closed
  on Unix systems, forcing any command that waits on stdin to
  immediately terminate. Defaults to false.


### stacktrace()


This function is deprecated. Use __STACKTRACE__ instead.
Deprecated mechanism to retrieve the last exception stacktrace.

It always return an empty list.


### stop(status \\ 0)
*(since 1.5.0)* 
```elixir
@spec stop(non_neg_integer() | binary()) :: :ok
```

Asynchronously and carefully stops the Erlang runtime system.

All applications are taken down smoothly, all code is unloaded, and all ports
are closed before the system terminates by calling `halt/1`.

`status` must be a non-negative integer or a binary.

- If an integer, the runtime system exits with the integer value which is
  returned to the operating system. On many platforms, only the status codes
  0-255 are supported by the operating system.

- If a binary, an Erlang crash dump is produced with status as slogan, and
  then the runtime system exits with status code 1.

Note this function is asynchronous and the current process will continue
executing after this function is invoked. In case you want to block the
current process until the system effectively shuts down, you can invoke
`Process.sleep(:infinity)`.

#### Examples

    System.stop(0)
    System.stop(1)


### system_time()

```elixir
@spec system_time() :: integer()
```

Returns the current system time in the `:native` time unit.

It is the VM view of the `os_time/0`. They may not match in
case of time warps although the VM works towards aligning
them. This time is not monotonic.

Inlined by the compiler.


### system_time(unit)

```elixir
@spec system_time(time_unit() | :native) :: integer()
```

Returns the current system time in the given time unit.

It is the VM view of the `os_time/0`. They may not match in
case of time warps although the VM works towards aligning
them. This time is not monotonic.


### time_offset()

```elixir
@spec time_offset() :: integer()
```

Returns the current time offset between the Erlang VM monotonic
time and the Erlang VM system time.

The result is returned in the `:native` time unit.

See `time_offset/1` for more information.

Inlined by the compiler.


### time_offset(unit)

```elixir
@spec time_offset(time_unit() | :native) :: integer()
```

Returns the current time offset between the Erlang VM monotonic
time and the Erlang VM system time.

The result is returned in the given time unit `unit`. The returned
offset, added to an Erlang monotonic time (for instance, one obtained with
`monotonic_time/1`), gives the Erlang system time that corresponds
to that monotonic time.


### tmp_dir()

```elixir
@spec tmp_dir() :: String.t() | nil
```

Writable temporary directory.

Returns a writable temporary directory.
Searches for directories in the following order:

1.  the directory named by the TMPDIR environment variable
2.  the directory named by the TEMP environment variable
3.  the directory named by the TMP environment variable
4.  `C:\TMP` on Windows or `/tmp` on Unix-like operating systems
5.  as a last resort, the current working directory

Returns `nil` if none of the above are writable.


### tmp_dir!()

```elixir
@spec tmp_dir!() :: String.t()
```

Writable temporary directory, exception on error.

Same as `tmp_dir/0` but raises `RuntimeError`
instead of returning `nil` if no temp dir is set.


### trap_signal(signal, id \\ make_ref(), fun)
*(since 1.12.0)* 
```elixir
@spec trap_signal(signal(), id, (-&gt; :ok)) ::
  {:ok, id} | {:error, :already_registered} | {:error, :not_sup}
when id: term()
```

Traps the given `signal` to execute the `fun`.

> #### Avoid setting traps in libraries {: .warning}
> 
> Trapping signals may have strong implications
> on how a system shuts down and behaves in production and
> therefore it is extremely discouraged for libraries to
> set their own traps. Instead, they should redirect users
> to configure them themselves. The only cases where it is
> acceptable for libraries to set their own traps is when
> using Elixir in script mode, such as in `.exs` files and
> via Mix tasks.

An optional `id` that uniquely identifies the function
can be given, otherwise a unique one is automatically
generated. If a previously registered `id` is given,
this function returns an error tuple. The `id` can be
used to remove a registered signal by calling
`untrap_signal/2`.

The given `fun` receives no arguments and it must return
`:ok`.

It returns `{:ok, id}` in case of success,
`{:error, :already_registered}` in case the id has already
been registered for the given signal, or `{:error, :not_sup}`
in case trapping exists is not supported by the current OS.

The first time a signal is trapped, it will override the
default behavior from the operating system. If the same
signal is trapped multiple times, subsequent functions
given to `trap_signal` will execute *first*. In other
words, you can consider each function is prepended to
the signal handler.

By default, the Erlang VM register traps to the three
signals:

- `:sigstop` - gracefully shuts down the VM with `stop/0`
- `:sigquit` - halts the VM via `halt/0`
- `:sigusr1` - halts the VM via status code of 1

Therefore, if you add traps to the signals above, the
default behavior above will be executed after all user
signals.

#### Implementation notes

All signals run from a single process. Therefore, blocking the
`fun` will block subsequent traps. It is also not possible to add
or remove traps from within a trap itself.

Internally, this functionality is built on top of `:os.set_signal/2`.
When you register a trap, Elixir automatically sets it to `:handle`
and it reverts it back to `:default` once all traps are removed
(except for `:sigquit`, `:sigterm`, and `:sigusr1` which are always
handled). If you or a library call `:os.set_signal/2` directly,
it may disable Elixir traps (or Elixir may override your configuration).


### unique_integer(modifiers \\ [])

```elixir
@spec unique_integer([:positive | :monotonic]) :: integer()
```

Generates and returns an integer that is unique in the current runtime
instance.

"Unique" means that this function, called with the same list of `modifiers`,
will never return the same integer more than once on the current runtime
instance.

If `modifiers` is `[]`, then a unique integer (that can be positive or negative) is returned.
Other modifiers can be passed to change the properties of the returned integer:

- `:positive` - the returned integer is guaranteed to be positive.
- `:monotonic` - the returned integer is monotonically increasing. This
  means that, on the same runtime instance (but even on different
  processes), integers returned using the `:monotonic` modifier will always
  be strictly less than integers returned by successive calls with the
  `:monotonic` modifier.

All modifiers listed above can be combined; repeated modifiers in `modifiers`
will be ignored.

Inlined by the compiler.


### untrap_signal(signal, id)
*(since 1.12.0)* 
```elixir
@spec untrap_signal(signal(), id) :: :ok | {:error, :not_found} when id: term()
```

Removes a previously registered `signal` with `id`.


### user_home()

```elixir
@spec user_home() :: String.t() | nil
```

User home directory.

Returns the user home directory (platform independent).


### user_home!()

```elixir
@spec user_home!() :: String.t()
```

User home directory, exception on error.

Same as `user_home/0` but raises `RuntimeError`
instead of returning `nil` if no user home is set.


### version()

```elixir
@spec version() :: String.t()
```

Elixir version information.

Returns Elixir's version as binary.




---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
