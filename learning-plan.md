# Rust Mastery Plan: The Ruby Systems Programmer's Path

## About This Plan

This plan is designed for a **senior Ruby/Rails developer** who knows Rust basics and wants to **master Rust and gain systems programming knowledge**. It uses a hybrid approach combining the best elements from four different pedagogical strategies:

- **Spiral structure**: Core concepts are revisited at increasing depth across 3 levels, mirroring how real understanding develops
- **Real projects**: Every module builds toward a named, usable tool -- not toy exercises
- **Ruby bridging**: Each module starts with what you already know from Ruby and shows where Rust diverges
- **Systems knowledge**: Every chapter includes a "Under the Hood" section explaining what the machine is actually doing

**Environment:** macOS, Ghostty terminal + Zellij, Neovim editor

**Estimated timeline:** ~45-60 sessions (1-2 hours each), across several months

---

## How This Plan Works

### The Spiral
We visit the same core concepts 3 times, each time going deeper:

| Concept | Spiral 1: Foundations | Spiral 2: Depth | Spiral 3: Mastery |
|---|---|---|---|
| **Ownership** | Moves, `Clone`/`Copy`, stack vs heap | `Rc`, `Arc`, interior mutability | `Pin`, custom smart pointers, arenas |
| **Borrowing** | `&T`, `&mut T`, basic rules | `Cow<T>`, reborrowing, borrow splitting | Stacked borrows, unsafe aliasing |
| **Lifetimes** | Elision, basic `'a` | Bounds on traits/structs, `'static` | HRTBs (`for<'a>`), variance |
| **Traits** | `impl Trait`, std traits | `dyn Trait`, vtables, orphan rules | Associated types, GATs, sealed traits |
| **Generics** | Basic `<T>`, trait bounds | `where`, `PhantomData`, const generics | Type-state, type-level programming |
| **Error Handling** | `Result`, `Option`, `?` | Custom errors, `thiserror`/`anyhow` | Error architecture for large systems |
| **Concurrency** | Basic threads, `Mutex`, channels | `Send`/`Sync`, `RwLock`, scoped threads | `async`/`await`, futures, executors |
| **Systems** | Stack vs heap, `Vec` internals | File descriptors, syscalls, buffered I/O | TCP/IP, mmap, cache-aware programming |

### Session Format
Each session follows this pattern:
1. Review where we left off (check progress tracking below)
2. Ruby bridge: connect new concepts to what you know
3. Systems knowledge: understand what the machine does
4. Hands-on coding: exercises and project work
5. Update progress tracking

### How to Pick Up Between Sessions
Check the **Progress Tracking** section at the bottom. It records exactly where you are, what you struggled with, and what comes next. Each module has clear entry and exit points.

---

## Spiral 1: Foundations (Modules 1-7)

**Goal:** Build a working mental model of Rust's core concepts. Write code that compiles. Understand *what* the rules are.

**Project arc:** Small CLI tools building toward `rgrep` -- a grep clone.

---

### Module 1: From `irb` to `cargo` -- Setting Up and First Programs (1-2 sessions)

**Ruby bridge:**
| Ruby | Rust | Notes |
|------|------|-------|
| `irb` / `pry` | `cargo run` (fast compile) | Rust is compiled; no eval loop |
| `bundle` / `gem` | `cargo` | Cargo = bundler + rake + gem combined |
| `Gemfile` / `Gemfile.lock` | `Cargo.toml` / `Cargo.lock` | Same dependency model |
| `rspec` | Built-in `#[test]` + `cargo test` | No separate framework needed |
| `rubocop` | `clippy` + `rustfmt` | Linting + formatting |

**Under the hood:**
Ruby is interpreted: write, run, see results. Rust is compiled: there's a compilation step between writing and running. This step is where Rust's power lives -- the compiler catches bugs that Ruby only finds at runtime. Compilation produces a native binary -- no interpreter, no VM, no GC overhead at runtime.

**Concepts:**
- Cargo: `new`, `build`, `run`, `test`, `clippy`, `rustfmt`
- `Cargo.toml` and dependencies
- Basic types: integers, floats, bools, `char`, `String`, `&str`
- Functions, variables, mutability
- Type inference (you write types less often than you'd think)

**Exercises:**
1. Create a project, write a function + test, run `cargo test`. Compare to Ruby workflow.
2. Add `serde` as a dependency. Compare `Cargo.toml` to `Gemfile`.
3. Intentionally write broken code. Read the compiler suggestions. Appreciate them.

**Mini-project: `rcat`**
A simplified `cat` command. Reads files passed as arguments, prints contents with optional line numbering (`-n`) and blank line squeezing (`-s`). Use `std::env::args()` and `std::fs`.

---

### Module 2: Types and Ownership -- The Concept Ruby Hides (2-3 sessions)

**Ruby bridge:**
In Ruby, you never think about who "owns" data. You create objects, pass them around, GC cleans up. Multiple variables reference the same object: `a = [1,2,3]; b = a` -- both point to the same array, mutations through `b` affect `a`.

In Rust:
```rust
let a = vec![1, 2, 3];
let b = a;          // MOVED. a is now invalid.
// println!("{:?}", a);  // COMPILE ERROR
```

The three ownership rules:
1. Each value has exactly **one owner**
2. When the owner goes out of scope, the value is **dropped**
3. Ownership can be **transferred** (moved) or **borrowed**

| Ruby | Rust | Key Difference |
|------|------|----------------|
| Integer (arbitrary size) | `i8`/`i16`/`i32`/`i64` | You choose the size |
| String (mutable, heap) | `String` (owned) vs `&str` (borrowed) | Two string types! |
| `nil` | `Option<T>` | **Biggest shift**: absence is explicit |
| Array | `Vec<T>` (homogeneous) | One type per collection |
| Hash | `HashMap<K, V>` | Fixed key and value types |

**Under the hood:**
Everything in memory is bytes. An `i32` is 4 bytes. A `String` is a `(pointer, length, capacity)` triple on the stack pointing to heap-allocated UTF-8 bytes. When a `String` is moved, only the 24-byte stack triple is copied -- the heap data stays put. When the owner is dropped, the heap data is freed. No GC needed, no reference counting overhead. This is why Rust is as fast as C while being memory-safe.

**Concepts:**
- Primitive types and their sizes (`std::mem::size_of`)
- `String` vs `&str` at the memory level
- Ownership: moves, `Copy` vs `Clone`
- `Option<T>` eliminates nil bugs
- `Vec<T>` growth strategy (pointer + length + capacity)
- Stack vs heap allocation

**Exercises:**
1. Write a program printing sizes of all primitive types. Compare to Ruby where everything is a heap object.
2. Create a `String`, move it, try to use the original. Read the error. Fix with `.clone()`.
3. Create a struct with `Drop` implementation. Observe drop order in nested scopes.
4. Create a `Vec<String>`, push items, print pointer/length/capacity after each push. Observe reallocation.

**Mini-project: Hex Dump Tool**
Build a hex dump tool (like `xxd`). Reads a file and prints contents in hex + ASCII format, 16 bytes per line, with offset addresses. Reinforces byte-level thinking and file I/O.

```
00000000: 4865 6c6c 6f2c 2077 6f72 6c64 210a       Hello, world!.
```

---

### Module 3: Borrowing and References -- Lending Without Losing (2-3 sessions)

**Ruby bridge:**
In Ruby, passing an object to a method gives that method access to the *same* object. No distinction between "giving" and "lending." Everything is a reference, always.

```ruby
def append(arr)
  arr.push("hello")  # mutates the ORIGINAL
end
```

Rust forces you to be explicit:
```rust
fn print_list(list: &Vec<String>) { /* read-only borrow */ }
fn append(list: &mut Vec<String>) { /* mutable borrow */ }
```

Borrowing rules:
1. **Many** `&T` (shared/immutable) OR **one** `&mut T` (exclusive/mutable), never both
2. References must always be **valid** (no dangling pointers)

**Under the hood:**
In C, two pointers to the same data + mutation = bugs: iterator invalidation, data races, use-after-free. Rust's borrowing rules prevent ALL of these at compile time. The rule "one mutable XOR many immutable" directly prevents the aliasing + mutation combination that causes these bugs. Slices (`&[T]`, `&str`) are "fat pointers": a pointer plus a length, giving bounds-checked access to contiguous data.

**Concepts:**
- References `&T` and `&mut T` as pointers with guarantees
- The aliasing XOR mutation principle
- Slices: `&[T]`, `&str`
- Reborrowing basics
- Why these rules prevent real bugs (iterator invalidation, data races)
- Basic lifetime annotations (first contact, not mastery)

**Exercises:**
1. Rewrite Module 2 functions to borrow instead of taking ownership. See how the caller keeps using values.
2. Write 5 snippets that violate borrow rules. For each, explain what bug the compiler is preventing.
3. Implement `fn longest_word(text: &str) -> &str` and `fn second_largest(nums: &[i32]) -> Option<i32>`.
4. Port a Ruby class with mutating and reading methods to Rust with `&self` and `&mut self`.

**Mini-project: Text Statistics Tool**
Read a text file, compute: word count, line count, character count, top 10 most frequent words, average word length, longest line. Use `&str` slices to avoid unnecessary allocations. Compare with `wc` for simple counts.

---

### Module 4: Structs, Enums, and Pattern Matching (2-3 sessions)

**Ruby bridge:**
Ruby bundles data + behavior in classes. Rust splits them:
- **Structs** hold data (like `attr_reader` fields)
- **`impl` blocks** add behavior
- **Enums** encode variants (Ruby symbols on steroids -- variants can carry data!)

```ruby
# Ruby
class User
  attr_accessor :name, :role
  def admin?() = @role == :admin
end
```

```rust
// Rust
enum Role { Admin, Member, Guest { expires_at: u64 } }

struct User { name: String, role: Role }

impl User {
    fn is_admin(&self) -> bool { matches!(self.role, Role::Admin) }
}
```

No inheritance. No `method_missing`. No monkey-patching. Composition and traits replace them.

**Under the hood:**
Structs have padding and alignment constraints -- CPUs access memory most efficiently when data is aligned to size boundaries. Enums are "tagged unions": a discriminant tag + space for the largest variant. Rust optimizes brilliantly: `Option<&T>` uses the null pointer as `None`, so it's the same size as a raw pointer. Understanding layout helps write cache-friendly code.

**Concepts:**
- Structs (named, tuple, unit), `impl` blocks
- Enums with data, `Option<T>`, `Result<T, E>` as enums
- Pattern matching: `match`, `if let`, `while let`
- Exhaustiveness checking (compiler ensures all cases handled)
- Destructuring, match guards, nested patterns
- Struct layout, alignment, niche optimization

**Exercises:**
1. Model a card deck with enums (`Suit`, `Rank`) and `Card` struct. Implement `Display`.
2. Build an expression evaluator: `enum Expr { Num(f64), Add(Box<Expr>, Box<Expr>), ... }`. Parse and evaluate.
3. Implement a state machine for a TCP connection using enums. Enforce valid transitions.
4. Print sizes of `Option<Box<i32>>`, `Option<&i32>`, `Option<bool>`. Explain each.

**Mini-project: `todo-cli` -- Task Manager**
A command-line todo list. `Task` struct, `Priority` enum, `Status` enum. Commands: add, complete, list, filter by status/priority. Store as JSON using `serde_json`. Pattern matching for command dispatch.

---

### Module 5: Error Handling the Rust Way (1-2 sessions)

**Ruby bridge:**
Ruby uses exceptions -- invisible in type signatures, thrown from anywhere, caught anywhere:
```ruby
begin
  data = JSON.parse(File.read("config.json"))
rescue Errno::ENOENT => e
  puts "Not found"
rescue JSON::ParserError => e
  puts "Bad JSON"
end
```

Rust makes errors **explicit in the return type**:
```rust
fn read_config(path: &str) -> Result<Config, ConfigError> {
    let text = fs::read_to_string(path)?;  // ? propagates errors
    let config: Config = serde_json::from_str(&text)?;
    Ok(config)
}
```

**Under the hood:**
Ruby exceptions use stack unwinding -- expensive, invisible control flow. C uses error codes -- cheap but easy to ignore. Rust's `Result<T, E>` is a zero-cost enum: the `?` operator compiles to a simple branch. No hidden control flow, no forgotten error checks, no runtime cost beyond the branch. Panics exist for unrecoverable situations only.

**Concepts:**
- `Result<T, E>` and `Option<T>` combinators
- The `?` operator and early return
- Custom error types with enums
- `From` trait for error conversion
- `thiserror` for library errors, `anyhow` for application errors
- Panic vs Result: when to use which

**Exercises:**
1. Write a function that reads a config, parses integers, returns their sum. Chain IO/parse/validation errors with `?`.
2. Design a calculator error enum (division by zero, overflow, invalid input). Implement `Display` + `Error` manually, then with `thiserror`.
3. Navigate a nested `HashMap<String, HashMap<String, Vec<i32>>>` using only `Option` combinators (no `match`).

**Mini-project: Config Parser**
Build an INI/TOML-subset config parser with rich error reporting: file name, line number, column, helpful message. Library version with `thiserror`, CLI wrapper with `anyhow`. This teaches the library vs application error split.

---

### Module 6: Traits and Basic Generics (2-3 sessions)

**Ruby bridge:**
Ruby modules provide shared behavior through mixins:
```ruby
module Printable
  def print_formatted = puts "=== #{to_display} ==="
end
class User
  include Printable
  def to_display = "#{@name} <#{@email}>"
end
```

Rust traits are compile-time checked interfaces:
```rust
trait Printable {
    fn to_display(&self) -> String;
    fn print_formatted(&self) { println!("=== {} ===", self.to_display()); }
}
impl Printable for User {
    fn to_display(&self) -> String { format!("{} <{}>", self.name, self.email) }
}
```

Where Ruby says "if it quacks like a duck," Rust says "if it implements the `Quack` trait." Same idea, compile-time enforcement.

**Under the hood:**
Static dispatch (`impl Trait`, generics): the compiler generates specialized code for each concrete type. Zero runtime cost, enables inlining. Dynamic dispatch (`dyn Trait`): uses a vtable (table of function pointers) stored in a fat pointer (16 bytes on 64-bit). Prevents inlining, can cause cache misses. Choose static by default; dynamic when you need type erasure.

**Concepts:**
- Trait definition, implementation, default methods
- Trait bounds: `T: Trait`, `T: Trait1 + Trait2`
- `impl Trait` in argument and return position
- Standard traits: `Display`, `Debug`, `Clone`, `Default`, `PartialEq`, `Eq`, `Hash`, `Ord`
- Operator overloading, `Deref`/`DerefMut`
- Basic generics: `<T>`, `where` clauses
- Monomorphization (generics are zero-cost -- separate code per type)
- `dyn Trait` introduction (deep dive in Spiral 2)

**Exercises:**
1. Define `Shape` trait with `area()` + `perimeter()`. Implement for Circle, Rectangle, Triangle. Write functions using `impl Shape` and `&dyn Shape`.
2. Implement a generic `Stack<T>` with `push`, `pop`, `peek`. Add `Display` only when `T: Display`.
3. Use the type-state builder pattern: enforce at compile time that a URL is set before building an HTTP request.
4. Compare `&T` size vs `&dyn Trait` size. Explain why.

**Mini-project: Plugin System**
A text processor with a `Plugin` trait: `name()`, `process(&str) -> String`. Plugins: uppercase, word count, reverse, rot13. Registry stores `Box<dyn Plugin>`, applies plugins by name. Pipeline chains plugins. This teaches trait objects and the strategy pattern.

---

### Module 7: Iterators and Closures (1-2 sessions)

**Ruby bridge:**
This is where Ruby knowledge shines brightest:
```ruby
users.select { |u| u.active? }.map { |u| u.name.upcase }.sort.take(10)
```
```rust
users.iter().filter(|u| u.is_active()).map(|u| u.name.to_uppercase()).sorted().take(10).collect::<Vec<_>>()
```

Key differences: Rust iterators are **lazy** by default (like `Enumerator::Lazy`). You must `.collect()` to materialize. Closures have three types (`Fn`, `FnMut`, `FnOnce`) based on how they capture variables. And the whole chain compiles to a **single loop** with zero intermediate allocations.

**Under the hood:**
In Ruby, `.map.select.reduce` creates intermediate arrays at each step. In Rust, iterator chains compile down to one loop. Each adaptor is a struct holding the previous iterator, and `next()` calls inline into tight code. Closures are also zero-cost: a closure capturing `&x` is literally a struct containing `&x`, and calls inline completely.

**Concepts:**
- `Iterator` trait, `next()`, lazy evaluation
- Adaptors: `map`, `filter`, `take`, `skip`, `chain`, `zip`, `enumerate`, `flat_map`
- Consumers: `collect`, `fold`, `sum`, `any`, `all`, `find`
- Custom iterators
- Closures: capture by reference, mutable reference, `move`
- `Fn`, `FnMut`, `FnOnce`

**Exercises:**
1. Solve using only iterator chains: sum of squares of odd numbers; flatten `Vec<Vec<i32>>` and find 3 largest; zip two vectors and find pairs where sum > threshold.
2. Implement a `Fibonacci` iterator. Use it with `take`, `filter`, `sum`.
3. Write closures that capture by `&`, `&mut`, and `move`. Print sizes with `size_of_val`. Explain.

**Mini-project: Data Pipeline**
CLI tool that reads CSV from stdin and supports: `filter "age > 30"`, `sort "name"`, `head 10`, `count`, `sum "salary"`. Each transformation is an iterator adaptor. Composable, zero-cost processing.

---

### Spiral 1 Bridge Project: `rgrep` -- A Grep Clone (2-3 sessions)

**Integrates all Spiral 1 concepts.**

Build a fast, colorized grep alternative:
- Search for patterns (literal + regex with `regex` crate)
- Recursive directory search with `walkdir`
- Case-insensitive mode
- Context lines (-A, -B, -C flags)
- Colorized output with `colored` crate
- Respect `.gitignore` (simplified)
- Proper error handling with custom error types
- Clean CLI with `clap`

This is the acid test for Spiral 1. It requires ownership (file contents), borrowing (search results reference file data), error handling (IO errors, pattern errors), traits (Display for results), iterators (line processing), and pattern matching (command parsing).

---

## Spiral 2: Depth (Modules 8-14)

**Goal:** Understand *why* the rules exist. Learn the patterns Rustaceans use to work *with* the borrow checker. Start writing idiomatic Rust. Gain real systems knowledge.

**Project arc:** Building toward `rush` (a Unix shell) and `cask` (a key-value database).

---

### Module 8: Smart Pointers and Interior Mutability (2-3 sessions)

**Ruby bridge:**
Every Ruby object reference is essentially a smart pointer managed by the GC. `Rc<T>` is pure reference counting (like Ruby's GC but deterministic). `RefCell<T>` moves borrow checking to runtime -- the closest to Ruby's "everything is mutable" model.

| Ruby | Rust | Purpose |
|------|------|---------|
| GC handles sharing | `Rc<T>` | Shared ownership via reference counting |
| GIL handles threads | `Arc<T>` | Thread-safe shared ownership |
| Everything mutable | `RefCell<T>` | Runtime-checked mutability |
| Automatic | `Weak<T>` | Break reference cycles |

**Under the hood:**
`Rc<T>` stores a strong count + weak count alongside the data. When strong count hits zero, data is dropped immediately (deterministic, unlike GC). `Arc<T>` uses atomic operations for the count (more expensive due to CPU cache coherence protocols). `RefCell` maintains borrow state at runtime: panics if you violate the "one mutable XOR many immutable" rule. These are escape hatches, not defaults.

**Concepts:**
- `Box<T>` for heap allocation
- `Rc<T>` and `Arc<T>`: shared ownership
- `Cell<T>` and `RefCell<T>`: interior mutability
- `Cow<T>`: clone-on-write optimization
- `Weak<T>`: preventing cycles
- When to use which smart pointer

**Exercises:**
1. Build a graph with `Rc<RefCell<Node>>`. Traverse it. Create a cycle. Observe the memory leak. Fix with `Weak<T>`.
2. Implement a caching wrapper that computes a value lazily using `RefCell`. Then a thread-safe version with `Mutex`.
3. Write `fn process(input: &str) -> Cow<str>` that avoids allocation when input needs no changes.

**Mini-project: DOM Tree**
Model an HTML DOM tree. Nodes have parents (`Weak`) and children (`Rc<RefCell<Node>>`). Implement `querySelector`-style lookup, append/remove, and pretty printing. Reflect on why `Vec` is almost always better in practice (cache locality).

---

### Module 9: Lifetimes in Depth (2-3 sessions)

**Ruby bridge:**
Ruby doesn't have lifetimes because the GC tracks object liveness at runtime. Rust does it at compile time. Lifetime annotations don't change how long things live -- they prove to the compiler that references are valid.

**Under the hood:**
In C, returning a pointer to a local variable is use-after-free -- accessing freed memory causes garbage reads or crashes. Lifetimes are Rust's compile-time prevention: `'a` tells the compiler "this reference must not outlive scope `'a`." The three elision rules handle ~90% of cases automatically. Lifetime errors mean the compiler is saving you from a segfault.

**Concepts:**
- Lifetime annotations: `'a`, `'b`, `'static`
- Lifetime elision rules (the 3 rules)
- Multiple lifetimes and how they interact
- Lifetime bounds: `T: 'a`
- Structs holding references
- Self-referential structs (and why they're hard)
- Higher-ranked trait bounds (`for<'a>`) -- introduction

**Exercises:**
1. Given 5 function signatures with lifetime annotations, predict whether they compile. Verify.
2. Build a `Tokenizer<'a>` that borrows from source text and implements `Iterator` yielding `&'a str` tokens.
3. Implement `Merger<'a, 'b>` holding two slices with independent lifetimes. Explain why two lifetimes are needed.
4. Try creating a self-referential struct (String + &str into it). See why it fails. Solve with indices.

**Mini-project: Zero-Copy CSV Parser**
A CSV parser that borrows from the input string. `CsvParser::new(data: &str)`, `headers() -> Vec<&str>`, `rows() -> Vec<Vec<&str>>`, `get_column(name: &str) -> Vec<&str>`. All parsed data references the original string -- no copies.

---

### Module 10: File I/O and the Operating System (2-3 sessions)

**Ruby bridge:**
Ruby's `File.read`, `File.open`, `IO.readlines` abstract away the OS. Rust gives you control over buffering, file descriptors, and system calls -- the things Ruby hides.

**Under the hood:**
When you open a file, the OS kernel returns a file descriptor (fd) -- a small integer. Every read/write is a system call (~100ns overhead). Ruby's IO is always buffered. Rust's `File` is unbuffered by default -- every `read` is a syscall. `BufReader` accumulates reads into a buffer (default 8KB), making fewer syscalls. This can be 10-100x faster for small reads. stdin/stdout/stderr are fd 0, 1, 2.

**Concepts:**
- `File`, `BufReader`, `BufWriter`
- `Read`, `Write`, `Seek` traits
- `Path` and `PathBuf`
- Directory traversal with `read_dir`
- File permissions and metadata
- Memory-mapped files with `memmap2`
- File descriptors under the hood

**Exercises:**
1. Read a large file byte-by-byte: unbuffered vs `BufReader` vs 64KB buffer. Time each. The difference is dramatic.
2. Implement a recursive directory walker with glob patterns (simplified `find`).
3. Open several files, read `/dev/fd/` to list all open file descriptors.

**Mini-project: File Sync Tool**
A one-way file synchronizer (simplified `rsync`). Compares source and destination by size + modification time. Copies new/modified files. Dry-run mode. Progress display. Uses buffered I/O for efficient copying.

---

### Module 11: Modules, Crates, and the Build System (1-2 sessions)

**Ruby bridge:**
| Ruby | Rust |
|------|------|
| `require "library"` | `use crate::module::Type;` |
| Module (namespace) | `mod module_name { }` |
| `private` | Default (everything private unless `pub`) |
| `protected` | `pub(crate)`, `pub(super)` |
| `public` | `pub` |

Everything is private by default -- the opposite of Ruby.

**Under the hood:**
In Ruby, `require` loads and evaluates a file at runtime. In Rust, the compiler processes one crate at a time, producing machine code. The linker combines object files into a binary. Static linking (Rust's default) bundles everything into one self-contained binary. Understanding this explains why Rust binaries are large but portable, and why crate boundaries affect compile times.

**Concepts:**
- `mod`, `pub`, `use`, `pub(crate)`, `pub(super)`
- File structure conventions
- `Cargo.toml`, dependencies, workspaces
- Feature flags and `#[cfg()]`
- Build scripts, documentation, `cargo doc`
- Integration tests and doc tests

**Exercises:**
1. Restructure the CSV parser from Module 9 as a proper library: public API, internal parser module, error module, integration tests.
2. Add a `serde` feature flag. Use `#[cfg(feature = "serde")]` for conditional derives.
3. Create a workspace: library crate + CLI binary crate.

**Mini-project: `cargo-linecount`**
A cargo subcommand that counts lines of Rust code in a project, excluding tests and comments. Teaches Cargo tooling integration and project traversal.

---

### Module 12: Concurrency: Threads and Shared State (2-3 sessions)

**Ruby bridge:**
Ruby has threads but the GIL means only one thread runs Ruby code at a time. You've used threading for IO or relied on multi-process (Puma, Sidekiq). Rust gives you **true parallelism** with **compile-time safety**.

```rust
// True parallelism -- no GIL
let handles: Vec<_> = (0..10)
    .map(|i| thread::spawn(move || expensive_computation(i)))
    .collect();
```

The borrow checker IS the concurrency checker -- same rules, different context. If it compiles, no data races. Period.

**Under the hood:**
An OS thread has its own stack (~8KB-8MB) but shares the process heap. Context switches cost ~1-10 microseconds. Sharing mutable data without synchronization causes data races (undefined behavior). `Mutex` uses atomic CPU instructions (compare-and-swap) for locking. `RwLock` allows multiple readers OR one writer. `Send` means safe to transfer across threads; `Sync` means safe to share references across threads. `Rc` is not `Send` (non-atomic count); `Arc` is `Send` (atomic count).

**Concepts:**
- `std::thread::spawn`, `JoinHandle`
- `Mutex<T>`, `MutexGuard` (RAII locking)
- `RwLock<T>`
- `Arc<Mutex<T>>` pattern
- `Send` and `Sync` marker traits
- Scoped threads (`std::thread::scope`)
- Atomic types and basic ordering
- Channels (`mpsc`)
- Deadlock and prevention

**Exercises:**
1. Split a large vector into chunks, sum each in a thread, combine. Benchmark 1/2/4/8 threads.
2. Thread-safe counter three ways: `Mutex<i64>`, `AtomicI64`, channels. Benchmark under contention.
3. Create a deadlock. Fix with consistent lock ordering.

**Mini-project: Parallel Word Counter**
Given a directory of text files, count word frequencies across all files using threads. Compare: (a) single `Arc<Mutex<HashMap>>`, (b) per-thread maps merged at end, (c) channel-based. Benchmark all three.

---

### Module 13: Collections and Advanced Data Structures (1-2 sessions)

**Under the hood:**
Modern CPUs have cache hierarchy (L1: ~4 cycles, L2: ~12, L3: ~40, RAM: ~200 cycles). `Vec<T>` is cache-friendly (contiguous, prefetcher helps). `LinkedList` scatters nodes across heap (cache miss per access). This is why `Vec` beats linked lists for almost everything despite Big-O. `HashMap` uses SwissTable with SIMD-accelerated probing. `BTreeMap` stores multiple keys per node for cache efficiency.

**Concepts:**
- `HashMap`/`HashSet`: hashing, load factor, SwissTable
- `BTreeMap`/`BTreeSet`: sorted, cache-friendly
- `VecDeque`: ring buffer
- `BinaryHeap`: max-heap
- The `Entry` API for efficient map operations
- Cache-friendly vs cache-hostile layouts

**Exercises:**
1. Pathological hash experiment: implement `Hash` that always returns 0. Benchmark vs default.
2. Word frequency counter three ways: `contains_key + insert`, `or_insert`, `entry().and_modify().or_insert()`. Benchmark.
3. For 5 scenarios, choose the optimal collection and justify.

**Mini-project: LRU Cache**
LRU cache with O(1) get/put. `HashMap` for lookup + `VecDeque` for eviction ordering. Configurable capacity. Optional TTL.

---

### Module 14: Advanced Error Handling and API Design (1-2 sessions)

**Concepts deepened:**
- Error type design: library vs application
- Error composition with `From` conversions
- `thiserror` for structured library errors
- `anyhow` with `.context()` for application errors
- Retry patterns: transient vs permanent errors
- Backtraces and debugging

**Exercises:**
1. Design error hierarchy for a file processing library: `ParseError`, `IoError`, `ValidationError`.
2. Convert a program from `anyhow` to custom errors, and vice versa. Understand when each fits.
3. Implement `fn retry<T, E, F: FnMut() -> Result<T, E>>(f: F, max: usize) -> Result<T, E>`.

**Mini-project: JSON Schema Validator**
Define schemas, parse JSON with `serde_json`, validate against schemas, return rich structured errors with paths to invalid values.

---

### Spiral 2 Bridge Project: `rush` -- A Unix Shell (3-4 sessions)

**Integrates all Spiral 1 + 2 concepts. Major systems programming project.**

Build a functional Unix shell:
1. **Session 1:** Basic REPL. Parse and execute simple commands with `std::process::Command`. `cd` as builtin.
2. **Session 2:** Piping (`ls | grep foo | wc -l`) and I/O redirection (`>`, `<`, `>>`). Tokenizer/parser.
3. **Session 3:** Environment variable expansion (`$HOME`). Builtins: `export`, `echo`, `pwd`, `exit`. Command history with `rustyline`.
4. **Session 4:** Signal handling (Ctrl+C). Background jobs (`&`). Basic job control. `.rushrc` config.

**Systems concepts exercised:** Fork/exec/wait, file descriptors, pipes, signals, TTY, process inheritance, PATH resolution, job control.

---

## Spiral 3: Mastery (Modules 15-20)

**Goal:** Push the boundaries. Async Rust, macros, unsafe, performance optimization. Build serious systems software.

**Project arc:** Building toward `prism` -- a Redis-compatible server.

---

### Module 15: Async/Await and Tokio (2-3 sessions)

**Ruby bridge:**
Ruby has Fibers for cooperative concurrency. Rust async is fundamentally different: `async fn` returns a `Future` that does nothing until `.await`ed. You need a runtime (`tokio`) to drive futures. Unlike Ruby's single-threaded async, Rust's is multi-threaded.

**Under the hood:**
Threads are expensive (~8KB-8MB stack per thread). For I/O-heavy workloads (web servers handling thousands of connections), async is more efficient. Instead of blocking a thread, you register interest with the OS (`kqueue` on macOS) and get notified when data is ready. Rust's `async/await` compiles into state machines -- each `.await` is a state transition. The `Future` trait is like `Iterator` but for async: `poll()` returns `Ready` or `Pending`.

**Concepts:**
- `async fn`, `.await`, `Future` trait
- `tokio` runtime: `#[tokio::main]`, `spawn`, `select!`
- Async channels: `tokio::sync::mpsc`, `broadcast`
- CPU-bound vs I/O-bound workloads
- When to use threads vs async
- `Pin` -- conceptual understanding (why it exists)

**Exercises:**
1. Fetch multiple URLs concurrently with `reqwest`. Compare sequential vs `tokio::join!`.
2. Implement a producer-consumer with async channels. Observe bounded channel backpressure.
3. Use `tokio::select!` for timeouts and multi-channel actors.

**Mini-project: Async Chat Server**
TCP chat server with `tokio`. Multiple clients, named users, rooms, private messages, `/list` command. Uses `tokio::sync::broadcast` for the message bus.

---

### Module 16: Networking from Scratch (2-3 sessions)

**Under the hood:**
TCP is a reliable, ordered byte stream on top of IP packets. A socket is an OS file descriptor. `bind()` assigns an address, `listen()` accepts connections, each `accept()` returns a new socket. TCP has no message boundaries -- you get a stream of bytes. You need framing: length-prefixed, delimiter-separated, or fixed-size messages.

**Concepts:**
- TCP sockets: `TcpListener`, `TcpStream`
- Protocol design: framing, serialization
- `serde` for serialization
- Connection handling patterns
- Graceful shutdown
- Partial reads and buffering

**Exercises:**
1. TCP echo server: single-threaded, then multi-threaded, then async.
2. Design a length-prefixed protocol: 4-byte header + JSON payload. Handle partial reads.
3. Implement a client-side connection pool.

**Mini-project: `hurl` -- HTTP Client**
HTTP/1.1 client built on raw `TcpStream`. Parse response status/headers/body manually. POST with JSON. Builder pattern for requests. Add HTTPS with `rustls`. CLI with colored output and timing.

---

### Module 17: Macros -- Declarative and Procedural (2-3 sessions)

**Ruby bridge:**
Ruby metaprogramming: `method_missing`, `define_method`, `class_eval`. Rust macros operate at compile time on the AST -- more restricted but no runtime cost and no surprises.

**Concepts:**
- `macro_rules!`: repetition, hygiene, token trees
- Proc macros: derive, attribute, function-like
- `syn` and `quote` crates
- When macros are appropriate vs generics/traits

**Exercises:**
1. `vec_of_strings!`, `hashmap!` literal syntax, `retry!` macro.
2. `#[derive(Builder)]` proc macro using `syn`/`quote`.
3. `#[timed]` attribute macro that prints function execution time.

**Mini-project: `weave` -- Macro Toolkit**
A derive macro library: `Builder` (auto-generate builder pattern), `EnumVariants` (generate `variants()` method). Publish as a workspace with proc-macro crate.

---

### Module 18: Unsafe Rust and FFI (2-3 sessions)

**Ruby bridge:**
Ruby C extensions and `ffi` gem let you call C. Rust's `unsafe` is like Ruby's `send(:private_method)` -- bypassing safety for a specific reason. `unsafe` doesn't turn off the borrow checker -- it unlocks 5 specific powers.

**Under the hood:**
The five `unsafe` superpowers: dereference raw pointers, call unsafe functions, access mutable statics, implement unsafe traits, access union fields. Good Rust uses `unsafe` internally to build safe abstractions (Vec uses raw pointers but exposes a safe API). FFI is inherently unsafe because the compiler can't verify C code.

**Concepts:**
- Raw pointers: `*const T`, `*mut T`
- The 5 unsafe superpowers
- Safe wrappers around unsafe code
- FFI: calling C from Rust, exposing Rust to C
- `bindgen`, `#[no_mangle]`, `extern "C"`
- Common UB: null dereference, data races, invalid references

**Exercises:**
1. Sum a `&[i32]` using raw pointer arithmetic (no indexing, no iterators).
2. Reimplement `split_at_mut` with unsafe. Explain why safe Rust can't express it.
3. Call `getpid()`, `gethostname()`, `getcwd()` via libc FFI.

**Mini-project: SQLite Bindings**
Minimal Rust bindings for SQLite via FFI. Wrap `sqlite3_open`, `sqlite3_exec`, `sqlite3_prepare_v2`, `sqlite3_step`, `sqlite3_column_*`, `sqlite3_close` in safe structs `Database` and `Statement`. Statement must not outlive Database (lifetime enforcement).

---

### Module 19: Performance, Profiling, and Optimization (2-3 sessions)

**Under the hood:**
Cache lines are 64 bytes. Sequential access (Vec iteration) is fast because the prefetcher loads ahead. Random access (pointer chasing) causes cache misses. Data-oriented design: arrange data for actual access patterns. Always measure first -- intuition about performance is often wrong.

**Concepts:**
- Benchmarking with `criterion`
- Flamegraphs with `cargo-flamegraph`
- CPU cache effects and data layout
- `#[inline]`, release profile tuning (`opt-level`, `lto`, `codegen-units`)
- Data-oriented design: SoA vs AoS
- SIMD basics

**Exercises:**
1. Write `criterion` benchmarks for a function from a previous module. Optimize and measure.
2. Sequential vs random access benchmark at varying data sizes (1KB to 100MB). Plot cache effects.
3. Profile a previous project with flamegraph. Identify and optimize hot spots.

**Mini-project: High-Performance Log Analyzer**
Analyze multi-GB log files. Parse entries, filter by date/level/pattern, aggregate stats (requests/sec, error rates, percentiles). Memory-mapped files, zero-copy parsing, parallel chunk processing. Target: 1GB in under 2 seconds.

---

### Module 20: Design Patterns and Architecture (1-2 sessions)

**Concepts:**
- Composition over inheritance (Rust's philosophy)
- Tower-style middleware pattern
- Entity-component-system (ECS) basics
- Plugin architectures with traits
- When to use enums vs trait objects vs generics
- Dependency injection in Rust

**Exercises:**
1. Observer pattern with trait objects, then with channels. Compare.
2. Middleware pipeline: `fn(Request) -> Response` with layers.
3. Plugin architecture with event registration.

**Mini-project: `mini-ecs`**
A tiny entity-component-system. Entities are IDs, components in typed arrays, systems query by component combination. Focus on API design and data layout.

---

### Spiral 3 Capstone: `prism` -- A Redis-Compatible Server (5-7 sessions)

**Integrates everything from all three spirals.**

A Redis-compatible server that handles real Redis clients:

1. **Session 1:** RESP protocol parser. Handle `PING`, `ECHO`, `SET`, `GET`. Accept connections with tokio.
2. **Session 2:** Data types: `LPUSH`/`LPOP`/`LRANGE` for lists, `HSET`/`HGET`/`HGETALL` for hashes. `DEL`, `EXISTS`, `TYPE`.
3. **Session 3:** `EXPIRE`, `TTL`. Passive expiry (check on access) + active expiry (background scan). `INCR`/`DECR`.
4. **Session 4:** Persistence: RDB-style snapshots + append-only file logging.
5. **Session 5:** Pub/sub: `SUBSCRIBE`, `PUBLISH` with `tokio::sync::broadcast`.
6. **Session 6:** Performance profiling and optimization. Run `redis-benchmark` against your server.
7. **Session 7:** Sets, sorted sets. `MULTI`/`EXEC` transactions. Graceful shutdown. Config file. Final polish.

**Rust concepts exercised:** Ownership, borrowing, lifetimes, traits, generics, async/await, tokio, error handling, serialization, unsafe (optional optimization), macros (optional command registration), performance tuning.

**Systems concepts exercised:** TCP/IP, protocol implementation, event-driven architecture, persistence strategies, pub/sub, timer management, memory management for long-running servers.

---

## Summary: Module Map

| # | Module | Sessions | Spiral | Key Focus |
|---|--------|----------|--------|-----------|
| 1 | Cargo & First Programs | 1-2 | 1 | Tooling, basic types |
| 2 | Types & Ownership | 2-3 | 1 | Ownership, moves, String vs &str |
| 3 | Borrowing & References | 2-3 | 1 | &T, &mut T, slices, borrow rules |
| 4 | Structs, Enums, Matching | 2-3 | 1 | Data modeling, pattern matching |
| 5 | Error Handling | 1-2 | 1 | Result, Option, ?, custom errors |
| 6 | Traits & Basic Generics | 2-3 | 1 | Interfaces, static/dynamic dispatch |
| 7 | Iterators & Closures | 1-2 | 1 | Zero-cost iteration, Fn traits |
| -- | **`rgrep` bridge project** | 2-3 | 1 | Integration of Spiral 1 |
| 8 | Smart Pointers | 2-3 | 2 | Rc, Arc, RefCell, Cow |
| 9 | Lifetimes in Depth | 2-3 | 2 | Annotations, bounds, variance |
| 10 | File I/O & OS | 2-3 | 2 | Syscalls, buffering, file descriptors |
| 11 | Modules & Crates | 1-2 | 2 | Visibility, workspaces, features |
| 12 | Concurrency | 2-3 | 2 | Threads, Mutex, channels, Send/Sync |
| 13 | Collections | 1-2 | 2 | HashMap, BTree, cache effects |
| 14 | Advanced Errors & API Design | 1-2 | 2 | Error architecture, thiserror/anyhow |
| -- | **`rush` bridge project** | 3-4 | 2 | Unix shell (integration of Spiral 2) |
| 15 | Async/Await & Tokio | 2-3 | 3 | Futures, tokio, async I/O |
| 16 | Networking | 2-3 | 3 | TCP, HTTP, protocol design |
| 17 | Macros | 2-3 | 3 | macro_rules!, proc macros, syn/quote |
| 18 | Unsafe & FFI | 2-3 | 3 | Raw pointers, C interop, safe wrappers |
| 19 | Performance | 2-3 | 3 | Profiling, caches, optimization |
| 20 | Design Patterns | 1-2 | 3 | Architecture, ECS, middleware |
| -- | **`prism` capstone** | 5-7 | 3 | Redis clone (integration of all) |

**Total: ~45-60 sessions across ~20 modules + 3 major projects**

---

## Quick Reference: When Stuck

| Feeling | Action |
|---|---|
| "The borrow checker won't let me" | Clone first, make it work, then optimize away clones |
| "I don't know which lifetime" | Start with elided (`'_`), add annotations only when compiler asks |
| "Generics or trait objects?" | Generics by default (faster). Trait objects for heterogeneous collections |
| "Error types are getting complex" | `anyhow` in binaries, `thiserror` in libraries |
| "I don't understand Pin" | Come back later. Not needed until async or self-referential structs |
| "Should I use unsafe?" | Almost never. Check for a safe crate first |

---

## Ruby to Rust Quick Reference

| Ruby | Rust | Chapter |
|------|------|---------|
| `nil` | `Option<T>` | 2 |
| `begin/rescue` | `Result<T, E>` + `?` | 5 |
| `.each`/`.map`/`.select` | `.iter().for_each()`/`.map()`/`.filter()` | 7 |
| Blocks / Procs / Lambdas | Closures (`Fn`, `FnMut`, `FnOnce`) | 7 |
| Duck typing | Trait bounds (`T: Display`) | 6 |
| `Class` | `struct` + `impl` | 4 |
| Inheritance | Traits (composition over inheritance) | 6 |
| `module` (mixin) | `trait` (with default methods) | 6 |
| `module` (namespace) | `mod` | 11 |
| GC | Ownership + borrowing | 2, 3 |
| `Mutex` (with GIL) | `Mutex<T>` (true parallelism!) | 12 |
| `Thread.new` | `std::thread::spawn` | 12 |
| Gems | Crates | 11 |
| `attr_accessor` | `pub` fields + methods | 4 |
| `method_missing` | No equivalent (and that's good) | -- |
| Monkey patching | Extension traits (controlled) | 6 |
| C extensions | FFI + `unsafe` | 18 |

---

## Progress Tracking

### Current Status
- **Current Module:** Not started
- **Current Spiral:** 1 (Foundations)
- **Sessions Completed:** 0
- **Last Session Date:** N/A
- **Next Step:** Begin Module 1

### Session Log

*(Updated after each session)*

| Session | Date | Module | Topics Covered | Exercises Done | Struggles | Key Insights |
|---------|------|--------|----------------|----------------|-----------|-------------|
| -- | -- | -- | -- | -- | -- | -- |

### Areas Needing Extra Practice

*(Updated based on struggles)*

### Completed Projects

| Project | Status | Notes |
|---------|--------|-------|
| `rcat` (Module 1) | Not started | -- |
| Hex Dump (Module 2) | Not started | -- |
| Text Stats (Module 3) | Not started | -- |
| `todo-cli` (Module 4) | Not started | -- |
| Config Parser (Module 5) | Not started | -- |
| Plugin System (Module 6) | Not started | -- |
| Data Pipeline (Module 7) | Not started | -- |
| `rgrep` (Bridge 1) | Not started | -- |
| DOM Tree (Module 8) | Not started | -- |
| CSV Parser (Module 9) | Not started | -- |
| File Sync (Module 10) | Not started | -- |
| `cargo-linecount` (Module 11) | Not started | -- |
| Parallel Word Counter (Module 12) | Not started | -- |
| LRU Cache (Module 13) | Not started | -- |
| JSON Validator (Module 14) | Not started | -- |
| `rush` (Bridge 2) | Not started | -- |
| Chat Server (Module 15) | Not started | -- |
| `hurl` (Module 16) | Not started | -- |
| `weave` (Module 17) | Not started | -- |
| SQLite Bindings (Module 18) | Not started | -- |
| Log Analyzer (Module 19) | Not started | -- |
| `mini-ecs` (Module 20) | Not started | -- |
| `prism` (Capstone) | Not started | -- |
