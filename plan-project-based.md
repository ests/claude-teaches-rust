# Rust Learning Plan: Project-Based Path

## Why Project-Based Learning Is the Best Way to Learn Rust

Rust is not a language you learn by reading about it. Its ownership model, borrow checker, and type system are concepts that only click when you fight with the compiler on real code. Abstract exercises ("implement a linked list") teach you to satisfy the compiler, but they don't teach you *why* Rust's constraints exist or *when* they help you.

Building real projects changes that equation completely:

**You learn concepts when you need them, not before.** When your CLI tool needs to handle errors from file I/O, network calls, and user input all at once, you'll *want* to understand `Result`, `?`, and custom error types -- because your program won't compile without them. This is fundamentally different from reading a chapter on error handling and doing exercises.

**You build intuition for ownership by feeling the pain.** The borrow checker isn't an obstacle -- it's preventing real bugs. When you're building a concurrent web server and the compiler refuses to let you share mutable state across threads, you'll understand *exactly* what data races look like and why `Arc<Mutex<T>>` exists. That lesson sticks in a way no textbook explanation can match.

**As a Rubyist, you'll appreciate the contrast.** Ruby gives you freedom and pays for it at runtime (nil errors, type mismatches, race conditions). Rust catches all of these at compile time. Building the same kinds of things you'd build in Ruby -- CLI tools, web services, data processors -- lets you feel this difference viscerally. You'll see what you gain and what you pay for it.

**Projects give you a portfolio and real tools.** At the end of this plan, you won't just "know Rust." You'll have built a grep clone, a shell, a key-value database, a web server, and more. These are things you can use, extend, and show to others.

**Systems knowledge comes naturally.** You can't build a shell without understanding processes and file descriptors. You can't build a database without understanding memory-mapped files and serialization. The systems knowledge isn't a separate curriculum -- it's woven into every project.

The plan below is ordered so each project builds on concepts from the previous one. Early projects are completable in 2-3 sessions. Later ones take 4-5. Between projects, short concept checkpoints solidify what you've learned before moving on.

---

## Project 1: `rumble` -- A Markdown Note-Taking CLI (Sessions 1-3)

### What You'll Build
A command-line tool for managing markdown notes. Create, list, search, tag, and display notes stored as files on disk. Think of it as a tiny, fast alternative to a notes app, entirely from the terminal.

### Rust Concepts
- Project structure with Cargo (workspace, crates, modules)
- Ownership and borrowing fundamentals (passing strings around)
- `String` vs `&str` -- the most common Rubyist confusion
- Structs and impl blocks (coming from Ruby classes)
- `enum`, `Option`, `Result` for error handling
- Pattern matching with `match`
- The `?` operator for error propagation
- Iterators and closures (`.filter()`, `.map()`, `.collect()`)
- Traits: `Display`, `FromStr`, `Debug`

### Systems Concepts
- File system operations (create, read, write, list directories)
- File metadata (timestamps, permissions)
- Environment variables (`HOME`, `XDG_DATA_HOME`)
- Command-line argument parsing
- Exit codes and stderr vs stdout

### Step-by-Step Build Plan
1. **Session 1:** Set up project with `clap` for arg parsing. Implement `rumble new "My Note"` that creates a markdown file with YAML frontmatter (title, date, tags). Implement `rumble list` to show all notes.
2. **Session 2:** Add `rumble search <query>` with basic substring matching. Add `rumble show <id>` with terminal-friendly markdown rendering. Add `rumble tag <id> <tag>` to manage tags.
3. **Session 3:** Add `rumble edit <id>` that opens `$EDITOR`. Implement fuzzy search. Add color output with `colored` crate. Write tests for core logic.

### Extension Ideas
- Export notes to HTML
- Sync notes between directories
- Add `rumble daily` for daily journal entries
- Implement note linking (wiki-style `[[references]]`)

---

## Concept Checkpoint 1: Ownership Deep-Dive

Before moving on, solidify ownership with targeted exercises:
- Write a function that takes `&str` and one that takes `String` -- understand when to use which
- Create a struct that borrows data vs one that owns data -- see lifetime annotations appear
- Implement `Clone` vs using references -- understand the performance trade-off
- Compare with Ruby: what happens when you pass a string to a method in Ruby vs Rust?

---

## Project 2: `rgrep` -- A Grep Clone with Superpowers (Sessions 4-6)

### What You'll Build
A fast, colorized `grep` alternative that searches files recursively, supports regex, respects `.gitignore`, and outputs results with context lines. This is your first taste of why Rust is used for CLI tools (ripgrep, the real tool, is written in Rust).

### Rust Concepts
- Lifetimes in practice (search results referencing file contents)
- Generics (`fn search<P: AsRef<Path>>`)
- Trait objects vs generics (dynamic vs static dispatch)
- Error handling with `thiserror` and custom error types
- The `Read` and `BufRead` traits
- Iterators: chaining, lazy evaluation, `peekable()`
- Testing: unit tests, integration tests, test fixtures

### Systems Concepts
- Buffered I/O vs unbuffered I/O (why `BufReader` matters)
- Memory-mapped files (introduction)
- File system traversal and symlinks
- Regular expressions at the systems level
- Piping and Unix philosophy (stdout for data, stderr for messages)

### Step-by-Step Build Plan
1. **Session 4:** Basic single-file search with literal strings. Read file line-by-line with `BufReader`. Print matching lines with line numbers and color highlighting.
2. **Session 5:** Add recursive directory walking with `walkdir`. Add regex support with the `regex` crate. Implement `.gitignore` parsing (simplified). Add context lines (-A, -B, -C flags).
3. **Session 6:** Performance: add memory-mapped file reading with `memmap2`. Benchmark against system grep. Add parallel search with `rayon`. Write comprehensive tests.

### Extension Ideas
- Add `--replace` for find-and-replace across files
- Support searching inside compressed files
- Add file type filtering (like ripgrep's `--type`)
- Implement incremental search (interactive mode with live results)

---

## Project 3: `hurl` -- An HTTP Client Library and CLI (Sessions 7-9)

### What You'll Build
A simplified HTTP/1.1 client that makes requests over raw TCP sockets. No `reqwest` or `hyper` -- you'll implement the HTTP protocol yourself. Then wrap it in a nice CLI for making API requests (like a mini `curl` or `httpie`).

### Rust Concepts
- Enums with data (HTTP methods, status codes as rich types)
- The builder pattern (request builder)
- `impl Trait` in function signatures
- `From` and `Into` trait conversions
- String parsing and formatting
- `HashMap` for headers
- Serialization with `serde` and `serde_json`
- Modules and visibility (`pub`, `pub(crate)`)

### Systems Concepts
- TCP sockets and the network stack
- HTTP/1.1 protocol: request/response format, headers, chunked encoding
- DNS resolution basics
- TLS/SSL concepts (using `rustls` for HTTPS)
- Connection lifecycle and keep-alive

### Step-by-Step Build Plan
1. **Session 7:** Build raw HTTP/1.1 GET requests over `TcpStream`. Parse response status line and headers manually. Handle basic response bodies.
2. **Session 8:** Add POST with JSON bodies. Implement the builder pattern for constructing requests. Add header management. Parse chunked transfer encoding.
3. **Session 9:** Add HTTPS support with `rustls`. Build the CLI wrapper with colored output, JSON pretty-printing. Add request/response timing. Handle redirects.

### Extension Ideas
- Connection pooling and keep-alive
- Cookie jar support
- Request history and replay
- Save/load request collections (like Postman)

---

## Concept Checkpoint 2: Lifetimes and Generics

Solidify the trickiest Rust concepts before the projects get harder:
- Write a function that returns a reference -- feel lifetime elision rules
- Implement a generic cache struct `Cache<K, V>` with get/set
- Create a trait with a generic method and implement it for multiple types
- Understand `'static` lifetime and when/why it appears
- Compare: Ruby duck typing vs Rust trait bounds -- same idea, different enforcement

---

## Project 4: `rush` -- A Unix Shell (Sessions 10-13)

### What You'll Build
A functional Unix shell that can run commands, handle pipes, manage environment variables, support basic scripting, and do job control. This is where systems programming gets real.

### Rust Concepts
- Process management and `std::process::Command`
- Advanced enums for AST representation (command parsing)
- Recursive data structures (pipelines of pipelines)
- `Box` for heap allocation and recursive types
- Closures and higher-order functions
- Interior mutability with `RefCell` (for shell state)
- Drop trait for cleanup

### Systems Concepts
- Unix process model: fork, exec, wait
- File descriptors, pipes, and redirection (stdin/stdout/stderr)
- Signals (SIGINT, SIGTSTP, SIGCHLD)
- Environment variables and process inheritance
- PATH resolution and executable lookup
- Job control (foreground/background processes)
- The TTY and terminal control

### Step-by-Step Build Plan
1. **Session 10:** Basic REPL loop. Parse simple commands. Execute external commands with `Command`. Handle `cd` as a builtin.
2. **Session 11:** Implement piping (`ls | grep foo | wc -l`). Add I/O redirection (`>`, `<`, `>>`). Build a simple tokenizer/parser.
3. **Session 12:** Add environment variable expansion (`$HOME`). Implement builtins: `export`, `echo`, `pwd`, `exit`. Add command history with `rustyline`.
4. **Session 13:** Signal handling (Ctrl+C doesn't kill the shell). Background jobs (`&`). Basic job control (`jobs`, `fg`, `bg`). Add `.rushrc` config file support.

### Extension Ideas
- Tab completion for commands and file paths
- Syntax highlighting for input
- Basic scripting: if/else, loops, variables
- Prompt customization with git branch display

---

## Project 5: `cask` -- A Key-Value Store with Bitcask Architecture (Sessions 14-17)

### What You'll Build
A persistent key-value database inspired by Bitcask (the storage engine behind Riak). Append-only log files, in-memory index, crash recovery, and compaction. This is a real database engine.

### Rust Concepts
- Traits for abstraction (define a `Storage` trait)
- Smart pointers: `Box`, `Rc`, `Arc`
- `HashMap` internals and custom hashing
- Serialization/deserialization with `serde` + `bincode`
- File I/O: seeking, writing at offsets
- The `Drop` trait for resource cleanup
- `impl Iterator` for scan operations
- Comprehensive error handling with custom error enums

### Systems Concepts
- Append-only log-structured storage
- Write-ahead logging concepts
- File system: fsync, durability guarantees
- Memory-mapped I/O for the index
- Compaction and garbage collection
- Crash recovery and data integrity (CRC checksums)
- Benchmarking and performance measurement

### Step-by-Step Build Plan
1. **Session 14:** Design the on-disk format (key length, value length, timestamp, key, value). Implement append-only writes to a log file. Build the in-memory `HashMap` index pointing to file offsets.
2. **Session 15:** Implement reads (look up offset in index, seek in file, read value). Add delete (tombstone markers). Implement startup recovery (rebuild index by replaying the log).
3. **Session 16:** Add log compaction (merge old segments, remove deleted/overwritten keys). Implement CRC checksums for data integrity. Add `fsync` for durability options.
4. **Session 17:** Build a CLI and a simple TCP protocol for client-server mode. Add range scans. Benchmark: measure ops/sec for reads and writes. Compare with Redis on simple benchmarks.

### Extension Ideas
- Transactions (batch writes)
- TTL (time-to-live) for keys
- Replication to a second node
- Write a simple Redis-compatible protocol layer

---

## Concept Checkpoint 3: Concurrency Foundations

Before entering concurrent/async territory:
- Spawn threads and share data with `Arc<Mutex<T>>` -- understand why `Mutex` alone isn't enough
- Use channels (`mpsc`) to pass messages between threads
- Implement a thread pool from scratch (simplified)
- Understand `Send` and `Sync` traits -- what makes a type thread-safe?
- Compare: Ruby GIL vs Rust's compile-time thread safety

---

## Project 6: `swarm` -- A Multi-Threaded Web Scraper (Sessions 18-20)

### What You'll Build
A concurrent web scraper that crawls websites, respects `robots.txt`, manages a URL frontier with multiple worker threads, and stores results. Rate limiting, deduplication, and polite crawling included.

### Rust Concepts
- `std::thread` and thread spawning
- `Arc<Mutex<T>>` for shared mutable state
- Channels (`mpsc`) for worker communication
- Thread pools with `rayon` or hand-rolled
- `Atomic` types for lock-free counters
- Lifetimes with threads (`'static` bound and `move` closures)
- The `Send` and `Sync` marker traits

### Systems Concepts
- Thread scheduling and OS threads vs green threads
- Mutex contention and deadlock avoidance
- Rate limiting and backpressure
- URL parsing and normalization
- robots.txt protocol
- Concurrent data structures (lock-free vs locked)

### Step-by-Step Build Plan
1. **Session 18:** Single-threaded crawler: fetch a page, parse links with `scraper` crate, add to a queue. Implement URL normalization and deduplication with `HashSet`.
2. **Session 19:** Make it multi-threaded: URL frontier as `Arc<Mutex<VecDeque>>`, worker threads pulling URLs. Add channels for results. Implement rate limiting per domain.
3. **Session 20:** Add `robots.txt` parsing and respect. Implement politeness (delay between requests to same domain). Store results (title, links, text) to the `cask` database from Project 5. Add progress reporting.

### Extension Ideas
- Async version with `tokio` (preview of Project 8)
- Export site maps
- Full-text search over crawled content
- Distributed crawling across multiple processes

---

## Project 7: `pine` -- A Static Site Generator (Sessions 21-23)

### What You'll Build
A fast static site generator (like Hugo or Jekyll, but yours). Markdown to HTML, templates, asset pipeline, live reload during development. As a Rails developer, you'll appreciate building the "framework" side.

### Rust Concepts
- Trait objects (`dyn Trait`) for plugin systems
- The newtype pattern for type safety
- `Cow<str>` for efficient string handling
- Builder pattern for configuration
- `std::path` and cross-platform file handling
- Parallel file processing with `rayon`
- Watching for file changes

### Systems Concepts
- File system watching (inotify/kqueue/FSEvents)
- Template engines and string interpolation at the systems level
- Asset hashing for cache busting
- Static file serving with HTTP
- Build systems and incremental compilation concepts

### Step-by-Step Build Plan
1. **Session 21:** Parse markdown files with frontmatter. Convert to HTML with `pulldown-cmark`. Apply a basic HTML template. Output to a `build/` directory.
2. **Session 22:** Add template inheritance (layouts, partials) with `tera`. Implement collections (blog posts sorted by date, tag pages). Add an asset pipeline (copy static files, hash for cache busting).
3. **Session 23:** Add `pine serve` with a built-in HTTP server and live reload (file watcher + WebSocket notification). Parallel builds with `rayon`. Add a plugin trait for custom processing steps.

### Extension Ideas
- Sass/SCSS compilation
- Image optimization
- RSS/Atom feed generation
- Deploy command (S3, GitHub Pages, Netlify)

---

## Concept Checkpoint 4: Async Rust Foundations

The hardest conceptual leap in Rust. Prepare before Project 8:
- Understand `Future` trait: what `poll()` does, what `Waker` is
- Compare: Ruby threads and fibers vs Rust async tasks
- Write a simple manual `Future` implementation
- Understand `Pin` at a conceptual level (why it exists, not every detail)
- Use `tokio::spawn` and `.await` -- feel the difference from threads

---

## Project 8: `rapid` -- An Async Web Framework (Sessions 24-28)

### What You'll Build
A web framework in the style of Axum/Actix-web, built from lower-level async primitives. HTTP server, router, middleware, extractors, JSON handling. This is the big one -- you'll understand what frameworks like Axum do under the hood.

### Rust Concepts
- Async/await in depth
- The `Future` trait and how the runtime works
- `tokio` runtime: tasks, spawning, select
- Async traits and the `async-trait` crate (or native async traits)
- Tower-style middleware (Service trait pattern)
- Type-state pattern for request pipeline
- Macros (derive macros for route handlers)
- Advanced generics and trait bounds

### Systems Concepts
- Event loops and epoll/kqueue
- Non-blocking I/O
- Connection handling: accept loop, per-connection tasks
- HTTP keep-alive and connection management
- Graceful shutdown
- Backpressure in async systems

### Step-by-Step Build Plan
1. **Session 24:** Build an async TCP listener with `tokio`. Accept connections, read HTTP requests, send back hardcoded responses. Handle multiple connections concurrently.
2. **Session 25:** Implement a router (method + path -> handler). Create the `Handler` trait. Support path parameters (`/users/:id`). Return proper HTTP responses with status codes and headers.
3. **Session 26:** Add JSON request/response with `serde`. Implement "extractors" (like Axum: handler parameters automatically parsed from request). Add query string parsing.
4. **Session 27:** Build a middleware system. Implement logging middleware, CORS middleware, and authentication middleware. Chain middleware in a pipeline.
5. **Session 28:** Add static file serving. Implement graceful shutdown. Add request timeout handling. Build a sample REST API (todo app or similar) using your framework. Benchmark with `wrk` or `hey`.

### Extension Ideas
- WebSocket support
- Server-sent events
- Database connection pooling (with `sqlx`)
- Template rendering integration
- OpenAPI spec generation

---

## Project 9: `tinydb` -- A SQL Database Engine (Sessions 29-33)

### What You'll Build
A simplified relational database that understands a subset of SQL. Tokenizer, parser, query planner, B-tree storage engine, and a wire protocol. This is serious systems programming.

### Rust Concepts
- Recursive descent parsing
- Algebraic data types for AST representation
- Complex enum matching and destructuring
- Unsafe Rust (for B-tree node manipulation -- your first encounter)
- Custom iterators for query execution (volcano model)
- Memory management: `Box`, `Vec` internals
- Trait-based abstraction layers

### Systems Concepts
- B-tree data structure and on-disk layout
- Page-based storage (4KB pages like real databases)
- Buffer pool management
- SQL parsing and query planning basics
- Write-ahead logging for crash recovery
- The database storage hierarchy

### Step-by-Step Build Plan
1. **Session 29:** Build a SQL tokenizer (SELECT, INSERT, CREATE TABLE, WHERE). Parse into an AST. Execute `CREATE TABLE` and `INSERT` with in-memory storage (`HashMap` of `Vec<Row>`).
2. **Session 30:** Implement `SELECT` with `WHERE` clause filtering. Add basic type system (INTEGER, TEXT, BOOLEAN). Implement `ORDER BY` and `LIMIT`.
3. **Session 31:** Replace in-memory storage with a B-tree. Implement page-based storage (fixed-size 4KB pages). Serialize/deserialize rows into pages.
4. **Session 32:** Persist the B-tree to disk. Implement a buffer pool (cache frequently accessed pages). Add crash recovery with a simple write-ahead log.
5. **Session 33:** Add a client-server mode with a simple wire protocol. Implement `UPDATE` and `DELETE`. Add basic indexing. Run benchmarks.

### Extension Ideas
- JOIN support (nested loop join, then hash join)
- Transactions with MVCC
- A simple query optimizer (cost-based)
- Compatible protocol with PostgreSQL wire format (so you can use `psql`)

---

## Concept Checkpoint 5: Unsafe Rust and FFI

After touching `unsafe` in the database project:
- Understand what `unsafe` blocks promise to the compiler
- Write safe abstractions over unsafe code
- Call a C library from Rust (FFI basics)
- Understand `*const T`, `*mut T`, and when to use them
- Compare: Ruby C extensions vs Rust FFI

---

## Project 10: `gossip` -- A Distributed Chat System (Sessions 34-37)

### What You'll Build
A peer-to-peer chat application using a gossip protocol. Nodes discover each other, messages propagate through the network, and the system handles node failures. Real distributed systems programming.

### Rust Concepts
- Advanced async patterns: `select!`, `timeout`, cancellation
- `tokio::sync` primitives: `broadcast`, `watch`, `Notify`
- Serialization for network protocols with `serde` + `bincode`
- State machines with enums
- Advanced error handling: retries, circuit breakers
- `Pin` and `Stream` trait for async iteration

### Systems Concepts
- Gossip protocols and epidemic algorithms
- UDP vs TCP trade-offs
- Network partitions and failure modes
- Consistent hashing for peer discovery
- Message ordering and vector clocks (simplified)
- NAT traversal concepts

### Step-by-Step Build Plan
1. **Session 34:** Build a simple UDP-based messaging system. Two peers can send messages directly. Serialize messages with `bincode`. Handle message framing.
2. **Session 35:** Implement peer discovery with a gossip protocol (each node periodically shares its peer list). Handle node joins and graceful departures.
3. **Session 36:** Implement reliable message broadcast (each node forwards messages it hasn't seen). Add message deduplication. Handle network partitions gracefully.
4. **Session 37:** Add a TUI (terminal UI) with `ratatui`. Show online peers, chat rooms, message history. Handle node failures and reconnection.

### Extension Ideas
- End-to-end encryption
- File transfer
- Persistent message history
- Leader election for coordination

---

## Project 11: `weave` -- A Rust Macro and Procedural Macro Toolkit (Sessions 38-40)

### What You'll Build
A collection of useful macros: a `derive(Builder)` macro (auto-generate builder pattern), a `#[route]` attribute macro (for the web framework), and a `sql!` macro for compile-time SQL validation. Learn the Rust metaprogramming system.

### Rust Concepts
- Declarative macros (`macro_rules!`)
- Procedural macros: derive, attribute, function-like
- Token streams and `syn`/`quote` crates
- The Rust compilation model
- Compile-time code generation
- Hygiene in macros

### Systems Concepts
- Metaprogramming: compile-time vs runtime code generation
- AST manipulation
- Code generation patterns in systems languages
- Comparison with Ruby metaprogramming (method_missing, define_method)

### Step-by-Step Build Plan
1. **Session 38:** Start with `macro_rules!`: build a `vec_of_strings!`, a `hashmap!` literal syntax, and a `retry!` macro with configurable attempts. Understand macro hygiene and repetition.
2. **Session 39:** Build a `derive(Builder)` proc macro. Parse struct fields with `syn`, generate builder methods with `quote`. Handle optional fields and validation.
3. **Session 40:** Build a `#[route("/path")]` attribute macro for annotating handler functions (tie back to Project 8). Build a `sql!("SELECT ...")` macro that validates SQL syntax at compile time.

### Extension Ideas
- A `#[cached]` macro for memoization
- A `#[timed]` macro for performance logging
- Domain-specific language (DSL) macros
- A derive macro for custom serialization

---

## Project 12: `forge` -- A Container Runtime (Sessions 41-45)

### What You'll Build
A simplified container runtime that can create isolated processes using Linux namespaces and cgroups (or simulated equivalents on macOS). Think of it as a tiny Docker. This is peak systems programming.

*Note: Full namespace support requires Linux. On macOS, we'll implement process isolation with what's available and use a Linux VM for the full experience.*

### Rust Concepts
- Unsafe Rust for system calls
- FFI with libc
- Raw pointers and memory safety at the boundary
- Error handling for OS-level operations
- File system manipulation at a low level
- Integration testing with system state

### Systems Concepts
- Linux namespaces (PID, NET, MNT, UTS, USER)
- cgroups for resource limits
- chroot and pivot_root for filesystem isolation
- Overlay filesystems
- Container images and layers
- The OCI runtime specification

### Step-by-Step Build Plan
1. **Session 41:** On macOS: implement process isolation with `sandbox-exec` profiles. On Linux: create a new PID namespace with `clone()`. Run a process in isolation. Set hostname with UTS namespace.
2. **Session 42:** Implement filesystem isolation. Create a root filesystem. Use `chroot` (or `pivot_root` on Linux) to give the process its own filesystem view.
3. **Session 43:** Add resource limits with cgroups (Linux) or process limits (macOS). Limit CPU, memory, and number of processes. Implement a simple image format (tar layers).
4. **Session 44:** Build a CLI: `forge run <image> <command>`. Pull a minimal image (Alpine/busybox). Implement layer caching. Add network isolation basics.
5. **Session 45:** Add a simple container registry protocol. Implement `forge build` with a Dockerfile-like format. Final integration: run a web server inside your container.

### Extension Ideas
- Container networking (virtual bridges)
- Volume mounts
- Multi-container orchestration
- Image building with layer caching

---

## Concept Checkpoint 6: Performance and Optimization

Final review before the capstone:
- Profile Rust programs with `flamegraph` and `perf`
- Understand zero-cost abstractions (look at assembly output)
- Memory allocators: global vs arena allocation
- SIMD basics
- Compare: Ruby vs Rust performance characteristics and when each wins

---

## Project 13: Capstone -- `prism` -- A Redis-Compatible In-Memory Data Store (Sessions 46-52)

### What You'll Build
A Redis-compatible server that handles real Redis clients. RESP protocol, multiple data types (strings, lists, hashes, sets, sorted sets), persistence (RDB snapshots + AOF), pub/sub, Lua scripting basics, and cluster mode. This is the culmination of everything -- networking, data structures, concurrency, persistence, and protocol implementation.

### Rust Concepts
- Everything from all previous projects, integrated
- Advanced trait patterns (trait inheritance, associated types)
- Custom allocators (arena allocation for command parsing)
- Lock-free data structures (for pub/sub)
- Advanced lifetime patterns
- Performance optimization and profiling
- Benchmarking with `criterion`

### Systems Concepts
- Protocol implementation (RESP - Redis Serialization Protocol)
- Event-driven architecture
- Memory management for a long-running server
- Persistence strategies: snapshotting vs append-only
- Pub/sub messaging patterns
- Timer management (TTL, expiry scanning)

### Step-by-Step Build Plan
1. **Session 46:** Implement RESP protocol parser (bulk strings, arrays, integers, errors). Handle `PING`, `ECHO`, `SET`, `GET` commands. Accept connections with tokio.
2. **Session 47:** Add data types: `LPUSH`/`LPOP`/`LRANGE` for lists, `HSET`/`HGET`/`HGETALL` for hashes. Implement `DEL`, `EXISTS`, `TYPE`.
3. **Session 48:** Add `EXPIRE`, `TTL`, `PEXPIRE`. Implement passive expiry (check on access) and active expiry (background scanning). Add `INCR`/`DECR` with atomic operations.
4. **Session 49:** Implement persistence: RDB-style snapshots (serialize state to disk on `SAVE`/`BGSAVE`). Add AOF (append-only file) logging for durability.
5. **Session 50:** Add pub/sub: `SUBSCRIBE`, `PUBLISH`, `UNSUBSCRIBE`. Use `tokio::sync::broadcast` channels. Handle pattern subscriptions.
6. **Session 51:** Performance optimization. Profile with `flamegraph`. Optimize hot paths. Run the Redis benchmark tool (`redis-benchmark`) against your server.
7. **Session 52:** Add sets and sorted sets. Implement `MULTI`/`EXEC` transactions. Final polish: config file, logging, graceful shutdown. Celebrate.

### Extension Ideas
- Lua scripting with `EVAL`
- Cluster mode with hash slots
- Redis Streams
- Client libraries in other languages that connect to your server

---

## Summary: Project Map

| # | Project | Sessions | Key Rust Concepts | Key Systems Concepts |
|---|---------|----------|-------------------|---------------------|
| 1 | `rumble` - Note CLI | 1-3 | Ownership, structs, enums, error handling | File I/O, CLI design |
| 2 | `rgrep` - Grep clone | 4-6 | Lifetimes, generics, traits, testing | Buffered I/O, regex |
| 3 | `hurl` - HTTP client | 7-9 | Builder pattern, serde, modules | TCP, HTTP protocol, TLS |
| 4 | `rush` - Unix shell | 10-13 | Box, closures, interior mutability | Processes, pipes, signals |
| 5 | `cask` - KV database | 14-17 | Smart pointers, custom iterators | Log-structured storage, crash recovery |
| 6 | `swarm` - Web scraper | 18-20 | Threads, Arc/Mutex, channels | Concurrency, rate limiting |
| 7 | `pine` - Site generator | 21-23 | Trait objects, Cow, rayon | File watching, build systems |
| 8 | `rapid` - Web framework | 24-28 | Async/await, tokio, advanced generics | Event loops, non-blocking I/O |
| 9 | `tinydb` - SQL database | 29-33 | Parsing, unsafe, complex enums | B-trees, pages, WAL |
| 10 | `gossip` - Chat system | 34-37 | Advanced async, streams, state machines | Gossip protocol, UDP, failure handling |
| 11 | `weave` - Macro toolkit | 38-40 | Macros, proc macros, syn/quote | Metaprogramming, code generation |
| 12 | `forge` - Containers | 41-45 | Unsafe, FFI, raw pointers | Namespaces, cgroups, isolation |
| 13 | `prism` - Redis clone | 46-52 | Everything integrated | Everything integrated |

**Total: ~52 sessions, 13 projects, 6 concept checkpoints**

Each session is roughly 1-2 hours of focused work. The whole plan is designed to take you from "I know some Rust basics" to "I can build production-quality systems software in Rust" over several months of consistent practice.
