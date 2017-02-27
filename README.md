# Go Profiling
Just a collection of tools and commands to help profile and find areas to improve on in your Go code.

## Test profiling
- `go test -race`
    - Runs the tests with the [race detector](https://golang.org/doc/articles/race_detector.html) activated.
- `go test -benchmem`
    - Prints allocation statistics during benchmarks(number of bytes, and number of allocations).
- `go test -blockprofile out`
    - Writes a profile to out containing goroutine blocking statistics for use with `go tool pprof`.
- `go test -cpuprofile out`
    - Writes a profile to out containing CPU statistics for use with `go tool pprof`.
- `go test -memprofile out`
    - Writes a profile to out containing memory statistics for use with `go tool pprof`.
- `go test -mutexprofile out`
    - Writes a profile to out containing mutex contention statistics for use with `go tool pprof`.
- `go test -trace out`
    - Writes a profile to out containing the execution trace of the tests for use with `go tool pprof`.
- `go test -coverprofile out`
    - Writes a code coverage profile to out for use with `go tool cover -html`.

## Creating runtime profiling stats
- `GODEBUG="allocfreetrace=1,gctrace=1,scheddetail=1,schedtrace=X"`
    - Setting `$GODEBUG` makes the runtime print extra debugging information that can be useful for figuring out what's going on in at any moment.
    - `allocfreetrace=1` Prints every allocation and free including the related stack trace to stderr(this is a lot to print, so it'll be a good idea to redirect stderr to a file).
    - `gctrace=1` Prints every GC cycle and stats about the cycle(for more info read [here](https://golang.org/pkg/runtime/#hdr-Environment_Variables)).
    - `scheddetail=1,schedtrace=X` Every X milliseconds prints detailed information about the runtime(including goroutines, OS threads, and "processors"). For a quick overview, set `scheddetail=0`.
- `go build -race -gcflags "-m"`
    - `-race` builds the binary with the [race detector](https://golang.org/doc/articles/race_detector.html) activated.
    - `-gcflags "-m"` builds the binary with debug printing various information about optimizations as well as heap escapes and leaking params.
- [`net/http/pprof`](https://golang.org/pkg/net/http/pprof/)
    - Attaches a few routes under the `/debug/pprof/` endpoint that provide a profile that can be used with `go tool pprof`.
- [`net/http/httptrace`](https://golang.org/pkg/net/http/httptrace)
    - Allows you to trace http requests to determine connection reuse and dns lookup optimizations.

## Reading and visualizing runtime profiling stats
- `go tool pprof`
    - Runs the pprof cli for the given profile http endpoint or profile file, including the binary also includes specifics about each function.
- `go tool pprof -inuse_space`
    - Runs the same as the above except the profile must be a memory profile, and profiles the amount of memory in use.
- `go tool pprof -inuse_objects`
    - Runs the same as the above except the profile must be a memory profile, and profiles the number of objects used in memory.
- `go tool pprof -alloc_space`
    - Runs the same as the above except the profile must be a memory profile, and profiles the amount of memory allocated.
- `go tool pprof -alloc_objects`
    - Runs the same as the above except the profile must be a memory profile, and profiles the number of objects allocated.
- `go tool pprof -total_delay`
    - Runs the same as the above except the profile must be a blocking or mutex contention profile, and profiles the delay of contention points(blocking goroutines, mutex contention).
- `go tool pprof -contentions`
    - Runs the same as the above except the profile must be a blocking or mutex contention profile, and profiles the points of contention(blocking goroutines, mutex contention).
- `go tool trace`
    - Opens a browser window with links to various execution information gathered from the profile.
    - [View trace](http://127.0.0.1:50545/trace) displays a visual representation of the programs lifetime.
    - [Goroutine analysis](http://127.0.0.1:50545/goroutines) contains every goroutine and the time spent scheduling, in GC, networking, and code execution.
    - [Network blocking profile](http://127.0.0.1:50545/io) provides timing for every level of the network stack to see where blocking is occurring.
    - [Synchronization blocking profile](http://127.0.0.1:51903/block) provdes timing for every level of the stack showing where time is spent blocked during synchronization.
    - [Syscall blocking profile](http://127.0.0.1:51903/syscall) provides timing for syscalls to show where time is most spent in the kernel.
    - [Scheduler latency profile](http://127.0.0.1:51903/sched) provides timing for scheduler level information showing where time is most spent scheduling.
- [`github.com/davecheney/gcvis`](https://github.com/davecheney/gcvis)
    - Provides a visual representation of memory usage and the GC in real time.
- [`github.com/google/gops`](https://github.com/google/gops)
    - Provides access to print runtime information from Go processes by PIDs.
- [`github.com/derekparker/delve`](https://github.com/derekparker/delve)
    - Delve is a debugger providing a command line access as well as editor integrations.

## Benchmarking tools
- [`https://github.com/codesenberg/bombardier`](https://github.com/codesenberg/bombardier)
    - Is a HTTP benchmarking tool providing latency statistics, throughput, etc.
- [`golang.org/x/tools/cmd/benchcmp`](https://golang.org/x/tools/cmd/benchcmp)
    - Compares the output of two benchmarks from `go test` and displays the deltas between the two.
