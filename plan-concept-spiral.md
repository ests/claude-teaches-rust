# Rust Learning Plan: The Concept Spiral

## Why the Concept Spiral?

Rust is unlike most languages you've used. It has a steep learning curve not because any single concept is impossibly hard, but because **concepts are deeply interconnected** — ownership affects how you write functions, lifetimes affect how you design traits, traits affect how you structure generics, and all of it shapes how you think about error handling, concurrency, and performance.

The traditional "learn concept A, then B, then C" approach fails for Rust because:

1. **You can't understand lifetimes without first using borrowing in real code.** But you can't write interesting borrowed code without understanding some lifetimes. This chicken-and-egg problem exists for nearly every pair of Rust concepts.

2. **Premature depth kills momentum.** If you try to fully master ownership before moving on, you'll spend weeks on edge cases that won't make sense until you've seen traits and generics. You'll feel stuck and frustrated.

3. **Real understanding comes from revisiting concepts in new contexts.** The first time you use `&str` vs `String`, you learn the surface. The second time, in a struct with lifetimes, you understand *why*. The third time, building a zero-copy parser, you feel it in your bones.

The Concept Spiral works like this: **we visit the same core concepts 3-4 times, each time going deeper and connecting them to more of the language.** Each spiral level gives you a complete, working mental model — it's just that each successive model is more nuanced and powerful than the last.

This mirrors how experienced Rustaceans actually learned. Nobody masters lifetimes in one sitting. You develop intuition through repeated exposure at increasing depth, and suddenly the pieces click.

**For a Rubyist specifically**, this approach has another advantage: Ruby lets you hold many concepts loosely (duck typing, GC, dynamic dispatch). Rust requires you to hold them precisely. The spiral gives you time to shift your mental models gradually instead of all at once.

---

## Concept Progression Matrix

This matrix shows how the same core concepts deepen across spiral levels:

| Concept | Spiral 1: Foundations | Spiral 2: Depth | Spiral 3: Mastery | Spiral 4: Expert |
|---|---|---|---|---|
| **Ownership & Moves** | Basic moves, `Clone` vs `Copy`, stack vs heap | Interior mutability (`Cell`, `RefCell`), `Rc`, `Arc` | Self-referential structs, `Pin`, custom smart pointers | `MaybeUninit`, placement new patterns, arena allocators |
| **Borrowing & References** | `&T`, `&mut T`, borrowing rules | Reborrowing, borrow splitting, `Cow<T>` | Non-lexical lifetimes internals, two-phase borrows | Stacked borrows model, `unsafe` aliasing rules |
| **Lifetimes** | Elision, basic annotations `'a` | Lifetime bounds on traits/structs, `'static` | Higher-ranked trait bounds (`for<'a>`), variance | Covariance/contravariance proofs, lifetime subtyping |
| **Traits** | `impl Trait`, common std traits (`Display`, `From`) | Trait objects (`dyn Trait`), orphan rules, supertraits | Associated types vs generics, trait coherence | Auto traits, marker traits, sealed traits, specialization |
| **Generics** | Basic `<T>`, trait bounds | `where` clauses, `PhantomData`, const generics | GATs, type-level programming | Monomorphization costs, conditional compilation |
| **Error Handling** | `Result`, `Option`, `?` operator | Custom error types, `thiserror`, error chains | `anyhow` vs `thiserror` design, `Error` trait | `no_std` error handling, panic hooks, catch_unwind |
| **Concurrency** | Basic threads, `Mutex`, channels | `Send`/`Sync`, `RwLock`, scoped threads | `async`/`await`, `Future` trait, executors | Lock-free data structures, `unsafe` concurrency, atomics ordering |
| **Memory & Systems** | Stack vs heap, `Box`, `Vec` internals | Allocator basics, alignment, `repr(C)` | Custom allocators, SIMD, cache-aware programming | FFI, `mmap`, zero-copy I/O, `no_std` |
| **Patterns & Architecture** | Enums as state, builder pattern | Typestate pattern, newtype, deref coercion | ECS architecture, plugin systems with traits | Compiler plugin patterns, proc macros |

---

## Spiral 1: Foundations (Modules 1-6)

**Goal:** Build a working mental model of Rust's core concepts. Write code that compiles. Understand *what* the rules are, even if *why* is still fuzzy.

**Coming from Ruby:** You'll notice Rust makes you explicit about things Ruby handles implicitly — memory, types, error cases. This spiral gets you comfortable with that explicitness.

### Module 1.1: Types, Ownership, and Your First Rust Programs

**Concepts introduced:** Ownership, moves, `Copy` types, stack vs heap, `String` vs `&str`
**Systems knowledge:** Stack frames, heap allocation, how memory actually works

**Exercises:**
1. Write a function that takes a `String`, modifies it, and returns it. Then refactor to use `&mut String`. Feel the difference.
2. Implement a simple `WordCounter` struct that takes text and counts word frequencies. Use `HashMap<String, usize>`.
3. Write a program that reads a file line-by-line and prints lines matching a pattern (baby `grep`). Handle the `String`/`&str` dance.

**Mini-project:** `rcat` — a simplified `cat` command that reads files, numbers lines, and optionally squeezes blank lines. Use `std::fs` and `std::io`.

**Ruby bridge:** In Ruby, `a = "hello"; b = a` gives you two references to the same object. In Rust, `let a = String::from("hello"); let b = a;` *moves* the value. `a` is gone. This is the fundamental shift.

---

### Module 1.2: Structs, Enums, and Pattern Matching

**Concepts introduced:** Structs, enums, `impl` blocks, pattern matching, `Option<T>`
**Concepts revisited:** Ownership (structs owning their fields), moves (matching moves values)

**Exercises:**
1. Model a deck of cards using enums (`Suit`, `Rank`) and a `Card` struct. Implement `Display` for pretty printing.
2. Build a `LinkedList<i32>` using `enum Node { Cons(i32, Box<Node>), Nil }`. Implement `push`, `pop`, `len`, and `Display`.
3. Write a config parser that reads `KEY=VALUE` lines and returns `HashMap<String, String>`. Use `Option` and pattern matching to handle malformed lines gracefully.

**Mini-project:** `todo-cli` — a command-line todo list manager. Store todos in a JSON file. Support `add`, `done`, `list`, `remove` commands. Use `serde_json` for serialization.

**Ruby bridge:** Rust enums are like Ruby's case/when on steroids. Where Ruby uses duck typing and `nil`, Rust uses enums and `Option`. The compiler forces you to handle every case.

---

### Module 1.3: Error Handling and the `Result` Type

**Concepts introduced:** `Result<T, E>`, `?` operator, custom error enums, `From` trait for error conversion
**Concepts revisited:** Enums (errors are enums), pattern matching (matching on errors), ownership (errors own their data)

**Exercises:**
1. Write a temperature converter (Celsius/Fahrenheit/Kelvin) that parses user input and returns `Result` for invalid input.
2. Build a CSV parser that reads a file and returns `Vec<HashMap<String, String>>`. Propagate IO errors and parse errors using `?`.
3. Refactor the CSV parser to use a custom `CsvError` enum with variants for `IoError`, `ParseError`, and `MissingColumn`. Implement `From<std::io::Error>` for your error type.

**Mini-project:** `rurl` — a minimal HTTP client (like a baby `curl`). Use `std::net::TcpStream` to make a raw HTTP/1.1 GET request, parse the response, and print the body. This introduces you to networking and forces real error handling.

**Ruby bridge:** Ruby uses exceptions (raise/rescue) — you can ignore them and your code still compiles. Rust's `Result` is a value you *must* handle. The `?` operator is Rust's version of "just propagate the error," similar to letting exceptions bubble up.

---

### Module 1.4: Traits and Basic Generics

**Concepts introduced:** Defining traits, implementing traits, `impl Trait` syntax, basic generics `<T>`, trait bounds
**Concepts revisited:** Ownership (generic types and ownership), error handling (`Display` for errors)

**Exercises:**
1. Define a `Summary` trait with a `summarize(&self) -> String` method. Implement it for `NewsArticle` and `Tweet` structs. Write a function `notify(item: &impl Summary)`.
2. Implement `PartialOrd` and `Ord` for a `Version` struct (major, minor, patch). Write a function that sorts a `Vec<Version>`.
3. Write a generic `max_by_key` function: `fn max_by_key<T, K: Ord>(items: &[T], key: fn(&T) -> K) -> Option<&T>`.

**Mini-project:** `kv-store` — an in-memory key-value store. Define a `Storage` trait with `get`, `set`, `delete`. Implement `HashMapStorage` and `BTreeMapStorage`. Write a REPL interface that accepts `GET key`, `SET key value`, `DEL key`.

**Ruby bridge:** Traits are Rust's answer to Ruby modules/mixins and duck typing. Where Ruby says "if it quacks like a duck," Rust says "if it implements the `Quack` trait." Same idea, but checked at compile time.

---

### Module 1.5: Lifetimes — First Contact

**Concepts introduced:** Lifetime annotations `'a`, lifetime elision rules, lifetime in function signatures
**Concepts revisited:** Borrowing (lifetimes are about borrowing), references (lifetimes track reference validity), structs (structs holding references)

**Exercises:**
1. Write a function `longest<'a>(x: &'a str, y: &'a str) -> &'a str` and understand why the annotation is needed. Try removing it and read the error.
2. Build a `TextExcerpt<'a>` struct that holds a `&'a str` reference to part of a larger text. Implement methods that return sub-slices.
3. Write a `split_at_word<'a>(text: &'a str, word: &str) -> Option<(&'a str, &'a str)>` function. Notice which parameter needs a lifetime and which doesn't.

**Mini-project:** `log-parser` — parse a log file into structured entries. The `LogEntry` struct should *borrow* from the original file contents (zero-copy parsing). Implement filtering and searching on the parsed entries.

**Ruby bridge:** Ruby doesn't have lifetimes because the garbage collector tracks object liveness at runtime. Rust does it at compile time. Lifetimes are the compiler's way of proving "this reference is still valid." You're trading runtime overhead for compile-time safety.

---

### Module 1.6: Iterators and Closures

**Concepts introduced:** `Iterator` trait, `map`/`filter`/`collect`, closures, `Fn`/`FnMut`/`FnOnce`
**Concepts revisited:** Ownership (closures capture by move or borrow), traits (Iterator is a trait), generics (iterator adapters are generic), lifetimes (iterators borrowing from collections)

**Exercises:**
1. Implement a `Fibonacci` struct that implements `Iterator`. Use it with `take`, `filter`, `sum`.
2. Write a pipeline: read a file, split into words, normalize case, count frequencies, sort by frequency, take top 10. Chain iterator methods.
3. Write a function that accepts a closure: `fn apply_transform(data: &mut [f64], transform: impl FnMut(&mut f64))`. Use it with different closures.

**Mini-project:** `data-pipeline` — a CSV processing tool. Read CSV, apply user-specified transformations (filter rows, transform columns, aggregate), write output. Design the pipeline as a chain of iterator adapters.

**Ruby bridge:** Rust iterators are very similar to Ruby enumerables (`map`, `select`/`filter`, `reduce`/`fold`). The big difference: Rust iterators are lazy and zero-cost. No intermediate arrays are allocated. And closures explicitly capture variables (by reference or by move) rather than closing over everything implicitly.

---

### Bridge Project 1: `minigrep`

**Integrates:** All Spiral 1 concepts

Build an enhanced `grep` clone:
- Search for patterns in files (basic regex support using the `regex` crate)
- Support recursive directory search
- Case-insensitive mode
- Line numbers and context lines (like `grep -C`)
- Colorized output
- Read config from environment variables
- Proper error handling with custom error types
- Clean CLI with `clap`

This project forces you to connect ownership, borrowing, error handling, traits, iterators, and lifetimes in a real program. It's the acid test for Spiral 1.

---

## Spiral 2: Depth (Modules 2.1-2.6)

**Goal:** Understand *why* the rules exist. Learn the patterns Rustaceans use to work *with* the borrow checker instead of fighting it. Start writing idiomatic Rust.

**Coming from Ruby:** This is where you stop translating Ruby patterns into Rust and start thinking in Rust. The designs you reach for will change.

### Module 2.1: Smart Pointers and Interior Mutability

**Concepts deepened:** Ownership (shared ownership with `Rc`/`Arc`), borrowing (runtime borrow checking with `RefCell`), memory (heap allocation patterns)
**New concepts:** `Box<T>`, `Rc<T>`, `Arc<T>`, `Cell<T>`, `RefCell<T>`, `Deref`/`DerefMut` traits

**Exercises:**
1. Build a tree structure using `Rc<RefCell<Node>>`. Implement tree traversal (DFS, BFS).
2. Implement a simple caching layer: a struct that computes values lazily and caches them using `Cell` or `RefCell`.
3. Write a `SharedBuffer` backed by `Arc<Mutex<Vec<u8>>>` and use it across threads.

**Mini-project:** `dom-tree` — model an HTML DOM tree. Nodes have parents (weak references) and children (strong references). Implement `querySelector`-style lookup, tree manipulation (append, remove), and pretty-printing.

**Depth progression:** In Spiral 1, you learned "there's one owner." Now you learn "...unless you explicitly opt into shared ownership, and here's the cost."

---

### Module 2.2: Advanced Traits and Trait Objects

**Concepts deepened:** Traits (object safety, orphan rules, supertraits), generics (static vs dynamic dispatch)
**New concepts:** `dyn Trait`, vtables, trait object safety, associated types, blanket implementations

**Exercises:**
1. Define a `Drawable` trait hierarchy: `Drawable` (supertrait requiring `Debug`), with implementations for `Circle`, `Rectangle`, `Group` (which contains `Vec<Box<dyn Drawable>>`).
2. Implement a plugin system: define a `Plugin` trait, load "plugins" (structs implementing the trait) into a `Vec<Box<dyn Plugin>>`, and execute them.
3. Write a generic sorting function that works with any type implementing a custom `Sortable` trait. Then create a version using trait objects. Benchmark both.

**Mini-project:** `shape-renderer` — a 2D shape renderer that outputs SVG. Use trait objects for a heterogeneous collection of shapes. Implement transformations (translate, rotate, scale) using the newtype pattern.

**Depth progression:** In Spiral 1, you used `impl Trait` for static dispatch. Now you learn when and why to use `dyn Trait` instead, and the real cost of each approach.

---

### Module 2.3: Concurrency Fundamentals

**Concepts deepened:** Ownership (`Send`/`Sync` as ownership contracts), smart pointers (`Arc<Mutex<T>>`), traits (`Send` and `Sync` are auto traits)
**New concepts:** `std::thread`, `Mutex`, `RwLock`, channels (`mpsc`), scoped threads, `Send`/`Sync` marker traits

**Exercises:**
1. Write a parallel word counter: split a large file into chunks, count words in each chunk using threads, merge results.
2. Build a thread pool that accepts closures and distributes work across N worker threads using channels.
3. Implement a concurrent `HashMap` with `RwLock` that allows multiple readers or one writer. Benchmark against `Mutex<HashMap>`.

**Mini-project:** `web-crawler` — a concurrent web crawler. Use a thread pool to fetch pages in parallel. Use channels to communicate between the coordinator and worker threads. Track visited URLs. Respect a configurable concurrency limit.

**Depth progression:** In Spiral 1, you learned basic threads and `Mutex`. Now you understand *why* `Mutex<T>` returns a guard (RAII), why `Send`/`Sync` exist, and how Rust prevents data races at compile time.

---

### Module 2.4: Lifetime Patterns and Borrow Checker Mastery

**Concepts deepened:** Lifetimes (in structs, traits, impls), borrowing (`Cow<T>`, borrow splitting)
**New concepts:** Lifetime bounds on traits (`trait Foo<'a>`), `'static`, `Cow<'a, T>`, reborrowing, NLL

**Exercises:**
1. Build a `Tokenizer<'a>` that borrows from source text, yields tokens as `&'a str` slices, and implements `Iterator`.
2. Implement a function that accepts either owned or borrowed strings using `Cow<str>`. Benchmark to show when `Cow` avoids allocation.
3. Write a struct `Graph<'a>` where nodes borrow labels from an external string pool. Implement graph traversal methods with correct lifetime annotations.

**Mini-project:** `markdown-parser` — a subset Markdown parser that returns an AST borrowing from the input text. Support headers, paragraphs, bold, italic, links, and code blocks. This is a real test of lifetime management.

**Depth progression:** In Spiral 1, lifetimes were annotations you added to make the compiler happy. Now you understand *what they mean* — they're proof to the compiler about reference validity — and you can design APIs that use them intentionally.

---

### Module 2.5: Advanced Error Handling and API Design

**Concepts deepened:** Error handling (error type design, `thiserror`/`anyhow`), traits (`Error` trait, `From` conversions)
**New concepts:** Error type design philosophy, library vs application error handling, `thiserror`, `anyhow`, `Display` for errors, backtraces

**Exercises:**
1. Design an error hierarchy for a file processing library: `ParseError`, `IoError`, `ValidationError`, each with structured data. Use `thiserror`.
2. Convert a program from `anyhow` to custom error types, and another from custom types to `anyhow`. Understand when each is appropriate.
3. Implement a `retry` function: `fn retry<T, E, F: FnMut() -> Result<T, E>>(f: F, max_retries: usize) -> Result<T, E>`. Handle transient vs permanent errors.

**Mini-project:** `json-validator` — a JSON schema validator. Define a schema DSL, parse JSON (using `serde_json`), validate against schemas, and return rich, structured errors with paths to the invalid values.

**Depth progression:** In Spiral 1, you made errors compile. Now you design error types that are informative, composable, and follow Rust ecosystem conventions.

---

### Module 2.6: The Module System, Crates, and Testing

**Concepts deepened:** All previous concepts applied to larger codebases
**New concepts:** Module tree, `pub`/`pub(crate)`, workspaces, `Cargo.toml` features, unit tests, integration tests, doc tests, benchmarks

**Exercises:**
1. Refactor a single-file program into a multi-module library with a binary crate. Practice visibility rules.
2. Write a library with a public API, documentation, doc tests, and examples. Run `cargo doc` and review the output.
3. Set up a Cargo workspace with 3 crates: a core library, a CLI binary, and a shared test utilities crate.

**Mini-project:** `task-runner` — a Makefile-like task runner. Define tasks in a TOML file (task name, command, dependencies). Resolve dependency order (topological sort). Execute tasks. Structure as a workspace: core library + CLI.

**Depth progression:** Spiral 1 was single-file programs. Now you learn to structure Rust projects the way the ecosystem expects, with proper module boundaries and testing.

---

### Bridge Project 2: `http-server`

**Integrates:** All Spiral 1 and 2 concepts

Build a multi-threaded HTTP/1.1 server from scratch (no frameworks):
- Parse HTTP requests manually (method, path, headers, body)
- Route requests to handlers using a `Router` with trait objects
- Thread pool for concurrent connection handling
- Static file serving with MIME type detection
- Middleware system (logging, timing) using trait composition
- Graceful shutdown using channels
- Custom error types for all failure modes
- Well-structured as a Cargo workspace (library + binary)
- Comprehensive test suite

This is a substantial project that ties together everything from Spirals 1 and 2.

---

## Spiral 3: Mastery (Modules 3.1-3.6)

**Goal:** Push the boundaries of the type system and runtime. Understand async Rust, advanced generics, macros, and unsafe. Write code that is both safe and performant.

**Coming from Ruby:** This is territory Ruby doesn't have. You're learning things that are unique to systems programming and Rust's type system.

### Module 3.1: Async/Await and Futures

**Concepts deepened:** Concurrency (async vs threads), traits (`Future` trait), ownership (pinning, `Pin<T>`)
**New concepts:** `async`/`await`, `Future` trait, `tokio` runtime, `Pin<T>`, `Unpin`, async streams

**Exercises:**
1. Rewrite the web crawler from Module 2.3 using `async`/`await` with `tokio`. Compare code structure and performance.
2. Implement an async rate limiter using `tokio::time`. Wrap an async function to limit it to N calls per second.
3. Write an async function that races multiple futures and returns the first success, handling errors from losers gracefully.

**Mini-project:** `async-chat` — a TCP chat server using `tokio`. Support multiple rooms, private messages, user nicknames. Use async streams for message broadcasting.

**Depth progression:** In Spiral 2, you did concurrency with threads. Now you learn async — a different concurrency model that Rust is uniquely good at because ownership + lifetimes prevent the bugs that plague async code in other languages.

---

### Module 3.2: Advanced Generics and Type-Level Programming

**Concepts deepened:** Generics (const generics, `PhantomData`), traits (associated types, GATs), lifetimes (higher-ranked trait bounds)
**New concepts:** Const generics, `PhantomData`, typestate pattern, GATs, `for<'a>` bounds, conditional trait implementations

**Exercises:**
1. Implement a `Matrix<T, const ROWS: usize, const COLS: usize>` with compile-time dimension checking for multiplication.
2. Build a type-safe state machine using the typestate pattern: `Connection<Disconnected>` -> `Connection<Connected>` -> `Connection<Authenticated>`. Disallow invalid transitions at compile time.
3. Implement a generic `LendingIterator` using GATs (or simulate it) where the yielded item borrows from the iterator.

**Mini-project:** `type-safe-builder` — a builder pattern library where the type system tracks which fields have been set. `Builder<Missing<Name>, Set<Age>>` can't call `.build()` until all required fields are `Set`. Use `PhantomData` and typestate.

**Depth progression:** In Spiral 1, generics were `<T>`. In Spiral 2, you added bounds. Now you use the type system as a logic engine — encoding invariants at compile time so invalid states are unrepresentable.

---

### Module 3.3: Macros — Declarative and Procedural

**Concepts deepened:** Traits (derive macros), module system (proc macro crates)
**New concepts:** `macro_rules!`, proc macros (derive, attribute, function-like), `syn`/`quote` crates, hygiene

**Exercises:**
1. Write a `vec_of_strings!` macro that converts a list of expressions to `Vec<String>`. Then a `hashmap!` macro for `HashMap` literals.
2. Create a `#[derive(Builder)]` proc macro that generates a builder pattern for any struct.
3. Write an attribute macro `#[timed]` that wraps a function body to print its execution time.

**Mini-project:** `mini-serde` — a simplified `#[derive(Serialize, Deserialize)]` for a custom format (e.g., a simple binary format). Implement the proc macros from scratch using `syn` and `quote`.

**Depth progression:** Macros are Rust's metaprogramming — similar in spirit to Ruby's `method_missing` and `define_method`, but operating at compile time on the AST.

---

### Module 3.4: Unsafe Rust and FFI

**Concepts deepened:** Memory (raw pointers, layout), ownership (when you're the owner now), lifetimes (you're responsible for correctness)
**New concepts:** `unsafe` blocks, raw pointers, `unsafe impl`, FFI (`extern "C"`), `#[repr(C)]`, `std::ffi`

**Exercises:**
1. Implement a safe `split_at_mut` equivalent using `unsafe`. Explain why the safe version can't express this with the borrow checker.
2. Write a Rust wrapper around a C library (e.g., `libc`'s `getenv`, `setenv`). Create safe Rust abstractions.
3. Implement a simple bump allocator using `unsafe`: allocate from a pre-allocated buffer, return references with correct lifetimes.

**Mini-project:** `safe-vec` — implement your own `Vec<T>` from scratch using `unsafe`. Handle allocation, reallocation, `Drop`, `Deref`, iteration, and growth strategy. Wrap it all in a safe API.

**Depth progression:** All of Rust's safety guarantees rest on `unsafe` foundations. Understanding `unsafe` means understanding *what the compiler actually guarantees* and *what you must guarantee yourself*.

---

### Module 3.5: Performance and Systems Programming

**Concepts deepened:** Memory (allocators, cache lines, alignment), concurrency (atomics, lock-free), ownership (zero-copy)
**New concepts:** `std::alloc`, custom allocators, `std::sync::atomic`, memory ordering, SIMD, profiling, benchmarking with `criterion`

**Exercises:**
1. Profile a slow program using `cargo flamegraph`. Identify hot spots and optimize.
2. Implement a lock-free stack using `AtomicPtr` and compare-and-swap. Test it under contention.
3. Write a zero-copy log line parser that returns structs with `&[u8]` slices into a memory-mapped file.

**Mini-project:** `fast-csv` — a high-performance CSV parser. Use memory-mapped files, SIMD for newline detection (optional), zero-copy field extraction, and parallel parsing. Benchmark against the `csv` crate.

**Depth progression:** This is where the "systems" in "systems programming" really kicks in. You're thinking about what the CPU does with your data, not just what the language does.

---

### Module 3.6: Design Patterns and Architecture in Rust

**Concepts deepened:** Traits (composition over inheritance), generics (abstracting over behavior), error handling (in large systems)
**New concepts:** Entity-component-system (ECS), event systems, `tower`-style middleware, dependency injection in Rust

**Exercises:**
1. Implement the Observer pattern in Rust using trait objects. Then implement it using channels. Compare.
2. Build a middleware pipeline: `fn(Request) -> Response` composed of layers that can modify the request/response. Use closures and trait objects.
3. Design a plugin architecture where plugins register handlers for events. Use `inventory` or `linkme` for automatic registration.

**Mini-project:** `mini-ecs` — a tiny entity-component-system. Entities are IDs, components are stored in typed arrays (`Vec<Option<T>>`), systems are functions that query for entities with specific component combinations. Focus on the API design.

**Depth progression:** You can now design large Rust systems. You understand the trade-offs between static and dynamic dispatch, between trait objects and enums, between channels and shared state.

---

### Bridge Project 3: `mini-redis`

**Integrates:** All Spiral 1, 2, and 3 concepts

Build a Redis-compatible server:
- RESP protocol parser (zero-copy where possible)
- Async I/O with `tokio`
- Core commands: GET, SET, DEL, EXPIRE, KEYS, SUBSCRIBE, PUBLISH
- Pub/sub using async broadcast channels
- Key expiration using a background task
- Persistence (append-only file for durability)
- Custom proc macro for command registration
- `unsafe` for a performance-critical data structure (e.g., an intrusive linked list for LRU)
- Full test suite including async tests
- Benchmarks with `criterion`

This is a serious project that demonstrates Rust mastery.

---

## Spiral 4: Expert (Modules 4.1-4.4)

**Goal:** Go beyond what most Rust programmers know. Understand the compiler's reasoning, contribute to ecosystem crates, write libraries that others depend on.

### Module 4.1: Advanced Async — Executors, Wakers, and Runtime Internals

**Concepts deepened:** Async (the machinery under `async`/`await`), traits (`Future`, `Wake`), unsafe (`RawWaker`)
**New concepts:** Manual `Future` implementation, `Waker`/`RawWaker`, building a minimal executor, `Pin` projections, async cancellation safety

**Exercises:**
1. Implement `Future` manually for a `Delay` struct that completes after a duration.
2. Build a minimal single-threaded async executor (event loop + waker + task queue).
3. Write a `select!`-like combinator from scratch using `Pin` and manual `Future` polling.

**Mini-project:** `mini-tokio` — a minimal async runtime with a reactor (epoll/kqueue), task scheduler, and timer. Support spawning tasks, sleeping, and basic TCP I/O.

---

### Module 4.2: Advanced Unsafe — Invariants, Soundness, and Miri

**Concepts deepened:** Unsafe (soundness proofs, aliasing rules), memory (stacked borrows, provenance)
**New concepts:** Soundness, stacked borrows model, `Miri`, `unsafe` invariant documentation, `Pin` guarantees, self-referential types

**Exercises:**
1. Find and fix a soundness bug in a provided `unsafe` code sample. Document the invariant it violates.
2. Implement an intrusive doubly-linked list using `unsafe`. Document all safety invariants. Test with Miri.
3. Build a self-referential struct using `Pin` and `unsafe`. Understand why `Pin` exists.

**Mini-project:** `arena-alloc` — a typed arena allocator. Allocate objects that live for the arena's lifetime. Return references with the arena's lifetime. Ensure soundness. Test with Miri.

---

### Module 4.3: Advanced Macros and Compile-Time Computation

**Concepts deepened:** Macros (complex proc macros), traits (derive macros for complex types)
**New concepts:** Token tree munching, `proc-macro2`, compile-time assertions, `build.rs` code generation, conditional compilation

**Exercises:**
1. Write a `#[derive(EnumVariants)]` macro that generates a `variants() -> &[&str]` method.
2. Build a compile-time regex validator: `regex!("pattern")` that validates the regex at compile time and returns a compiled `Regex`.
3. Write a `build.rs` script that generates Rust code from a schema file (e.g., protobuf-like definition to Rust structs).

**Mini-project:** `query-builder` — a proc macro that turns a SQL-like DSL into type-checked Rust code. `query!(SELECT name, age FROM users WHERE age > 30)` should generate compiled, type-safe query code.

---

### Module 4.4: `no_std`, Embedded, and Cross-Compilation

**Concepts deepened:** Memory (no allocator), error handling (no `String`), traits (`core` vs `std`)
**New concepts:** `#![no_std]`, `core` vs `alloc` vs `std`, embedded HALs, cross-compilation, linker scripts

**Exercises:**
1. Port a simple library to `no_std`. Replace `String` with `heapless::String`, `Vec` with `heapless::Vec`.
2. Write a `no_std` binary that runs on QEMU (ARM Cortex-M). Blink a virtual LED.
3. Implement a simple protocol parser that works in `no_std` with only stack allocation.

**Mini-project:** `embedded-sensor` — a `no_std` firmware for an emulated sensor device. Read sensor values, apply a moving average filter, and output over UART. Run on QEMU.

---

### Bridge Project 4: `systems-project` (Choose One)

Pick one capstone project that combines everything:

**Option A: `mini-db`** — A simple embedded database
- B-tree index on disk
- WAL (write-ahead log) for crash recovery
- MVCC for concurrent reads
- Simple SQL subset parser (using macros)
- `mmap` for I/O
- Full async API

**Option B: `mini-compiler`** — A compiler for a small language
- Lexer and parser (zero-copy, using lifetimes)
- AST with arena allocation
- Type checker using trait-based visitors
- Code generation to a simple VM or WASM
- Optimization passes
- REPL using async I/O

**Option C: `container-runtime`** — A minimal Linux container runtime
- Linux namespace setup (using FFI to libc)
- Filesystem overlay (mount namespaces)
- Process isolation (PID namespaces)
- Resource limits (cgroups)
- CLI interface
- Requires Linux (can use a VM on macOS)

---

## Session Tracking Template

Each session should be logged with:

```
### Session [N] — [Date]
**Spiral:** [Level] | **Module:** [Number]
**Topics covered:**
**Exercises completed:**
**Struggles/Questions:**
**Key insights:**
**Next session starts at:**
```

---

## Estimated Timeline

- **Spiral 1 (Foundations):** ~12-18 sessions (including bridge project)
- **Spiral 2 (Depth):** ~12-18 sessions (including bridge project)
- **Spiral 3 (Mastery):** ~15-20 sessions (including bridge project)
- **Spiral 4 (Expert):** ~10-15 sessions (including capstone)
- **Total:** ~50-70 sessions

This is a rough guide. Some modules will click fast (especially where Ruby intuitions transfer), others will need more time (lifetimes, async, unsafe). The pace should adapt to understanding, not a calendar.

---

## Quick Reference: When Stuck

| Feeling | Action |
|---|---|
| "The borrow checker won't let me do X" | Clone first, make it work, then optimize away clones |
| "I don't know which lifetime to use" | Start with `'_` (elided), add annotations only when the compiler asks |
| "Should I use trait objects or generics?" | Start with generics (simpler, faster). Switch to trait objects if you need heterogeneous collections |
| "This error type is getting complex" | Use `anyhow` in binaries, custom errors in libraries |
| "I don't understand Pin" | Come back to it later. You don't need Pin until async or self-referential structs |
| "When should I use unsafe?" | Almost never. If you think you need it, check if there's a safe crate first |
