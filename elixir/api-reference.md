# API Reference

## Modules
**[Access](Access.md)**
Key-based access to data structures.

**[Agent](Agent.md)**
Agents are a simple abstraction around state.

**[Application](Application.md)**
A module for working with applications and defining application callbacks.

**[ArgumentError](ArgumentError.md)**
An exception raised when an argument to a function is invalid.

**[ArithmeticError](ArithmeticError.md)**
An exception raised on invalid arithmetic operations.

**[Atom](Atom.md)**
Atoms are constants whose values are their own name.

**[BadArityError](BadArityError.md)**
An exception raised when a function is called with the wrong number of arguments.

**[BadBooleanError](BadBooleanError.md)**
An exception raised when an operator expected a boolean, but received something else.

**[BadFunctionError](BadFunctionError.md)**

**[BadMapError](BadMapError.md)**
An exception raised when something expected a map, but received something else.

**[BadStructError](BadStructError.md)**

**[Base](Base.md)**
This module provides data encoding and decoding functions
according to [RFC 4648](https://tools.ietf.org/html/rfc4648).

**[Behaviour](Behaviour.md)** *deprecated*
Mechanism for handling behaviours.

**[Bitwise](Bitwise.md)**
A set of functions that perform calculations on bits.

**[Calendar](Calendar.md)**
This module defines the responsibilities for working with
calendars, dates, times and datetimes in Elixir.

**[Calendar.ISO](Calendar.ISO.md)**
The default calendar implementation, a Gregorian calendar following ISO 8601.

**[Calendar.TimeZoneDatabase](Calendar.TimeZoneDatabase.md)**
This module defines a behaviour for providing time zone data.

**[Calendar.UTCOnlyTimeZoneDatabase](Calendar.UTCOnlyTimeZoneDatabase.md)**
Built-in time zone database that works only in the `Etc/UTC` timezone.

**[CaseClauseError](CaseClauseError.md)**
An exception raised when a term in a `case/2` expression
does not match any of the defined `->` clauses.

**[Code](Code.md)**
Utilities for managing code compilation, code evaluation, and code loading.

**[Code.Fragment](Code.Fragment.md)**
This module provides conveniences for analyzing fragments of
textual code and extract available information whenever possible.

**[Code.LoadError](Code.LoadError.md)**
An exception raised when a file cannot be loaded.

**[Collectable](Collectable.md)**
A protocol to traverse data structures.

**[CompileError](CompileError.md)**
An exception raised when there's an error when compiling code.

**[CondClauseError](CondClauseError.md)**
An exception raised when no clauses in a `cond/1` expression evaluate to a truthy value.

**[Config](Config.md)**
A simple keyword-based configuration API.

**[Config.Provider](Config.Provider.md)**
Specifies a provider API that loads configuration during boot.

**[Config.Reader](Config.Reader.md)**
API for reading config files defined with `Config`.

**[Date](Date.md)**
A Date struct and functions.

**[Date.Range](Date.Range.md)**
Returns an inclusive range between dates.

**[DateTime](DateTime.md)**
A datetime implementation with a time zone.

**[Dict](Dict.md)** *deprecated*
Generic API for dictionaries.

**[Duration](Duration.md)**
Struct and functions for handling durations.

**[DynamicSupervisor](DynamicSupervisor.md)**
A supervisor optimized to only start children dynamically.

**[Enum](Enum.md)**
Functions for working with collections (known as enumerables).

**[Enum.EmptyError](Enum.EmptyError.md)**
An exception that is raised when something expects a non-empty enumerable
but finds an empty one.

**[Enum.OutOfBoundsError](Enum.OutOfBoundsError.md)**
An exception that is raised when a function expects an enumerable to have
a certain size but finds that it is too small.

**[Enumerable](Enumerable.md)**
Enumerable protocol used by `Enum` and `Stream` modules.

**[ErlangError](ErlangError.md)**

**[Exception](Exception.md)**
Functions for dealing with throw/catch/exit and exceptions.

**[File](File.md)**
This module contains functions to manipulate files.

**[File.CopyError](File.CopyError.md)**
An exception that is raised when copying a file fails.

**[File.Error](File.Error.md)**
An exception that is raised when a file operation fails.

**[File.LinkError](File.LinkError.md)**
An exception that is raised when linking a file fails.

**[File.RenameError](File.RenameError.md)**
An exception that is raised when renaming a file fails.

**[File.Stat](File.Stat.md)**
A struct that holds file information.

**[File.Stream](File.Stream.md)**
Defines a `File.Stream` struct returned by `File.stream!/3`.

**[Float](Float.md)**
Functions for working with floating-point numbers.

**[Function](Function.md)**
A set of functions for working with functions.

**[FunctionClauseError](FunctionClauseError.md)**
An exception raised when a function call doesn't match any defined clause.

**[GenEvent](GenEvent.md)** *deprecated*
An event manager with event handlers behaviour.

**[GenServer](GenServer.md)**
A behaviour module for implementing the server of a client-server relation.

**[HashDict](HashDict.md)** *deprecated*
Tuple-based HashDict implementation.

**[HashSet](HashSet.md)** *deprecated*
Tuple-based HashSet implementation.

**[IO](IO.md)**
Functions handling input/output (IO).

**[IO.ANSI](IO.ANSI.md)**
Functionality to render ANSI escape sequences.

**[IO.Stream](IO.Stream.md)**
Defines an `IO.Stream` struct returned by `IO.stream/2` and `IO.binstream/2`.

**[IO.StreamError](IO.StreamError.md)**

**[Inspect](Inspect.md)**
The `Inspect` protocol converts an Elixir data structure into an
algebra document.

**[Inspect.Algebra](Inspect.Algebra.md)**
A set of functions for creating and manipulating algebra
documents.

**[Inspect.Error](Inspect.Error.md)**
Raised when a struct cannot be inspected.

**[Inspect.Opts](Inspect.Opts.md)**
Defines the options used by the `Inspect` protocol.

**[Integer](Integer.md)**
Functions for working with integers.

**[Kernel](Kernel.md)**
`Kernel` is Elixir's default environment.

**[Kernel.ParallelCompiler](Kernel.ParallelCompiler.md)**
A module responsible for compiling and requiring files in parallel.

**[Kernel.SpecialForms](Kernel.SpecialForms.md)**
Special forms are the basic building blocks of Elixir, and therefore
cannot be overridden by the developer.

**[Kernel.TypespecError](Kernel.TypespecError.md)**
An exception raised when there's an error in a typespec.

**[KeyError](KeyError.md)**
An exception raised when a key is not found in a data structure.

**[Keyword](Keyword.md)**
A keyword list is a list that consists exclusively of two-element tuples.

**[List](List.md)**
Linked lists hold zero, one, or more elements in the chosen order.

**[List.Chars](List.Chars.md)**
The `List.Chars` protocol is responsible for
converting a structure to a charlist (only if applicable).

**[Macro](Macro.md)**
Functions for manipulating AST and implementing macros.

**[Macro.Env](Macro.Env.md)**
A struct that holds compile time environment information.

**[Map](Map.md)**
Maps are the "go to" key-value data structure in Elixir.

**[MapSet](MapSet.md)**
Functions that work on sets.

**[MatchError](MatchError.md)**
An exception raised when a pattern match (`=/2`) fails.

**[MismatchedDelimiterError](MismatchedDelimiterError.md)**
An exception raised when a mismatched delimiter is found when parsing code.

**[MissingApplicationsError](MissingApplicationsError.md)**
An exception that is raised when an application depends on one or more
missing applications.

**[Module](Module.md)**
Provides functions to deal with modules during compilation time.

**[NaiveDateTime](NaiveDateTime.md)**
A NaiveDateTime struct (without a time zone) and functions.

**[Node](Node.md)**
Functions related to VM nodes.

**[OptionParser](OptionParser.md)**
Functions for parsing command line arguments.

**[OptionParser.ParseError](OptionParser.ParseError.md)**
An exception raised when parsing option fails.

**[PartitionSupervisor](PartitionSupervisor.md)**
A supervisor that starts multiple partitions of the same child.

**[Path](Path.md)**
This module provides conveniences for manipulating or
retrieving file system paths.

**[Port](Port.md)**
Functions for interacting with the external world through ports.

**[Process](Process.md)**
Conveniences for working with processes and the process dictionary.

**[Protocol](Protocol.md)**
Reference and functions for working with protocols.

**[Protocol.UndefinedError](Protocol.UndefinedError.md)**
An exception raised when a protocol is not implemented for a given value.

**[Range](Range.md)**
Ranges represent a sequence of zero, one or many, ascending
or descending integers with a common difference called step.

**[Record](Record.md)**
Module to work with, define, and import records.

**[Regex](Regex.md)**
Provides regular expressions for Elixir.

**[Regex.CompileError](Regex.CompileError.md)**
An exception raised when a regular expression could not be compiled.

**[Registry](Registry.md)**
A local, decentralized and scalable key-value process storage.

**[RuntimeError](RuntimeError.md)**
An exception for a generic runtime error.

**[Set](Set.md)** *deprecated*
Generic API for sets.

**[Stream](Stream.md)**
Functions for creating and composing streams.

**[String](String.md)**
Strings in Elixir are UTF-8 encoded binaries.

**[String.Chars](String.Chars.md)**
The `String.Chars` protocol is responsible for
converting a structure to a binary (only if applicable).

**[StringIO](StringIO.md)**
Controls an IO device process that wraps a string.

**[Supervisor](Supervisor.md)**
A behaviour module for implementing supervisors.

**[Supervisor.Spec](Supervisor.Spec.md)** *deprecated*
Outdated functions for building child specifications.

**[SyntaxError](SyntaxError.md)**
An exception raised when there's a syntax error when parsing code.

**[System](System.md)**
The `System` module provides functions that interact directly
with the VM or the host system.

**[System.EnvError](System.EnvError.md)**
An exception raised when a system environment variable is not set.

**[SystemLimitError](SystemLimitError.md)**
An exception raised when a system limit has been reached.

**[Task](Task.md)**
Conveniences for spawning and awaiting tasks.

**[Task.Supervisor](Task.Supervisor.md)**
A task supervisor.

**[Time](Time.md)**
A Time struct and functions.

**[TokenMissingError](TokenMissingError.md)**
An exception raised when a token is missing when parsing code.

**[TryClauseError](TryClauseError.md)**
An exception raised when a term in a `try/1` expression
does not match any of the defined `->` clauses in its `else`.

**[Tuple](Tuple.md)**
Functions for working with tuples.

**[URI](URI.md)**
Utilities for working with URIs.

**[URI.Error](URI.Error.md)**
An exception raised when an error occurs when a `URI` is invalid.

**[UndefinedFunctionError](UndefinedFunctionError.md)**
An exception raised when a function is invoked that is not defined.

**[UnicodeConversionError](UnicodeConversionError.md)**

**[Version](Version.md)**
Functions for parsing and matching versions against requirements.

**[Version.InvalidRequirementError](Version.InvalidRequirementError.md)**
An exception raised when a version requirement is invalid.

**[Version.InvalidVersionError](Version.InvalidVersionError.md)**
An exception raised when a version is invalid.

**[Version.Requirement](Version.Requirement.md)**
A struct that holds version requirement information.

**[WithClauseError](WithClauseError.md)**
An exception raised when a term in a `with/1` expression
does not match any of the defined `->` clauses in its `else`.




---
[Next Page →](changelog.md "Changelog for Elixir v1.18")

---
Built using [ExDoc](https://github.com/elixir-lang/ex_doc "ExDoc") (v0.36.1) for the [Elixir programming language](href="https://elixir-lang.org" "Elixir").
