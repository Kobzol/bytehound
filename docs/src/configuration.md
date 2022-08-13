# Configuring the profiler

The profiler is configured through environment variables; here's a list of all of the
supported environment variables that you can set.

### `MEMORY_PROFILER_OUTPUT`

*Default: `memory-profiling_%e_%t_%p.dat`*

A path to a file to which the data will be written to.

This environment variable supports placeholders which will be replaced at
runtime with the following:
   * `%p` -> PID of the process
   * `%t` -> number of seconds since UNIX epoch
   * `%e` -> name of the executable
   * `%n` -> auto-incrementing counter (0, 1, .., 9, 10, etc.)

### `MEMORY_PROFILER_LOG`

*Default: unset*

The log level to use; possible values:
   * `trace`
   * `debug`
   * `info`
   * `warn`
   * `error`

Unset by default, which disables logging altogether.

### `MEMORY_PROFILER_LOGFILE`

*Default: unset*

Path to the file to which the logs will be written to; if unset the logs will
be emitted to stderr (if they're enabled with `MEMORY_PROFILER_LOG`).

This supports placeholders similar to `MEMORY_PROFILER_OUTPUT` (except `%n`).

### `MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS`

*Default: `0`*

When set to `1` the profiler will cull temporary allocations
and omit them from the output.

Use this if you only care about memory leaks or you want
to do long term profiling over several days.

### `MEMORY_PROFILER_TEMPORARY_ALLOCATION_LIFETIME_THRESHOLD`

*Default: `10000`*

The minimum lifetime of an allocation, in milliseconds, to **not** be
considered a temporary allocation, and hence not get culled.

Only makes sense when `MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS` is turned on.

### `MEMORY_PROFILER_TEMPORARY_ALLOCATION_PENDING_THRESHOLD`

*Default: None*

The maximum number of allocations to be kept in memory when tracking which
allocations are temporary and which are not.

Every allocation whose lifetime hasn't yet crossed the temporary allocation interval
will be temporarily kept in a buffer, and removed from it once it either gets deallocated
or its lifetime crosses the temporary allocation interval.

If the number of allocations stored in this buffer exceeds the value set here the buffer will be
cleared and all of the allocations contained within will be written to disk, regardless of their lifetime.

Only makes sense when `MEMORY_PROFILER_CULL_TEMPORARY_ALLOCATIONS` is turned on.

### `MEMORY_PROFILER_GRAB_BACKTRACES_ON_FREE`

*Default: `1`*

When set to `1` the backtraces will be also be gathered when the memory is freed.

### `MEMORY_PROFILER_DISABLE_BY_DEFAULT`

*Default: `0`*

When set to `1` the tracing will be disabled be default at startup.

### `MEMORY_PROFILER_REGISTER_SIGUSR1`

*Default: `1`*

When set to `1` the profiler will register a `SIGUSR1` signal handler
which can be used to toggle (enable or disable) profiling.

### `MEMORY_PROFILER_REGISTER_SIGUSR2`

*Default: `1`*

When set to `1` the profiler will register a `SIGUSR2` signal handler
which can be used to toggle (enable or disable) profiling.

### `MEMORY_PROFILER_ENABLE_SERVER`

*Default: `0`*

When set to `1` the profiled process will start an embedded server which can
be used to stream the profiling data through TCP using `bytehound gather` and `bytehound-gather`.

This server will only be started when profiling is first enabled.

### `MEMORY_PROFILER_BASE_SERVER_PORT`

*Default: `8100`*

TCP port of the embedded server on which the profiler will listen on.

If the profiler won't be able to bind a socket to this port it will
try to find the next free port to bind to. It will succesively probe
the ports in a linear fashion, e.g. 8100, 8101, 8102, etc.,
up to 100 times before giving up.

Requires `MEMORY_PROFILER_ENABLE_SERVER` to be set to `1`.

### `MEMORY_PROFILER_ENABLE_BROADCAST`

*Default: `0`*

When set to `1` the profiled process will send UDP broadcasts announcing that
it's being profiled. This is used by `bytehound gather` and `bytehound-gather`
to automatically discover `bytehound` instances to which to connect.

Requires `MEMORY_PROFILER_ENABLE_SERVER` to be set to `1`.

### `MEMORY_PROFILER_WRITE_BINARIES_TO_OUTPUT`

*Default: `1`*

Controls whenever the profiler will embed the profiled application (and all of the libraries
used by the application) inside of the profiling data it writes to disk.

This makes it possible to later decode the profiling data without having to manually
hunt down the original binaries.

### `MEMORY_PROFILER_ZERO_MEMORY`

*Default: `0`*

Decides whenever `malloc` will behave like `calloc` and fill the memory it returns with zeros.

### `MEMORY_PROFILER_BACKTRACE_CACHE_SIZE_LEVEL_1`

*Default: `16384`*

Controls the size of the internal backtrace cache used to deduplicate emitted stack traces.

This is the size of the per-thread cache.

### `MEMORY_PROFILER_BACKTRACE_CACHE_SIZE_LEVEL_2`

*Default: `327680`*

Controls the size of the internal backtrace cache used to deduplicate emitted stack traces.

This is the size of the global cache.

### `MEMORY_PROFILER_GATHER_MMAP_CALLS`

*Default: `0`*

Controls whenever the profiler will also gather calls to `mmap` and `munmap`.

(Those are *not* treated as allocations and are only available under the `/mmaps` API endpoint.)

### `MEMORY_PROFILER_USE_SHADOW_STACK`

*Default: `1`*

Whenever to use a more intrusive, faster unwinding algorithm; enabled by default.

Setting it to `0` will on average significantly slow down unwinding. This option
is provided only for debugging purposes.

### `MEMORY_PROFILER_TRACK_CHILD_PROCESSES`

*Default: `0`*

If set to `1`, bytehound will also track the memory allocations of child processes spawned by the
profiled process.
