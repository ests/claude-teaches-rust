# Rust Through the Lens of Systems Programming

## Why Systems-First Is the Best Way to Learn Rust

Most Rust tutorials teach the language feature by feature: here's `let`, here's `match`, here's the borrow checker. You memorize rules, fight the compiler, and wonder *why* Rust is the way it is. This approach leads to frustration because Rust's design decisions feel arbitrary without context.

**The systems-first approach flips this entirely.** Instead of asking "what does Rust do?", we ask "what does the computer do?" -- and then see how Rust gives you precise control over it.

Here is why this works especially well for someone coming from Ruby:

1. **Ruby hides the machine. Rust reveals it.** In Ruby, you never think about where objects live in memory, how strings are laid out, or what happens when you pass data between threads. Rust forces you to think about all of this -- but only because *the machine* forces you to. Understanding the machine makes Rust's rules feel natural, not restrictive.

2. **The borrow checker is not a language quirk -- it is a memory safety guarantee.** Once you understand how stack frames are allocated and freed, why use-after-free is catastrophic, and how data races corrupt memory, the borrow checker stops being an obstacle and starts being a tool you appreciate.

3. **Systems knowledge compounds.** Every chapter builds your mental model of how computers work. By the end, you will not just know Rust -- you will understand memory allocators, file descriptors, system calls, CPU caches, and concurrency primitives. This knowledge transfers to every language and every debugging session for the rest of your career.

4. **Projects feel meaningful.** Instead of toy exercises, you build things that interact with the real machine: memory-mapped files, network servers, lock-free data structures. You can see and measure the effects of your code on the actual hardware.

**The path:** We start at the bottom -- bytes, memory, the stack -- and build upward. Each chapter introduces systems concepts first, then shows how Rust's features map onto them. Small exercises let you verify your understanding, and each chapter ends with a mini-project that puts it all together.

---

## Plan Overview

| # | Chapter | Difficulty | Sessions | Systems Focus |
|---|---------|-----------|----------|---------------|
| 1 | Bytes, Memory Layout, and Primitive Types | Beginner | 1-2 | Memory representation |
| 2 | The Stack: Functions, Ownership, and Scope | Beginner | 1-2 | Stack frames, RAII |
| 3 | The Heap: Boxes, Strings, and Vectors | Beginner | 1-2 | Heap allocation, pointers |
| 4 | References, Borrowing, and the Borrow Checker | Beginner-Intermediate | 2-3 | Pointer aliasing, memory safety |
| 5 | Structs, Enums, and Data Layout | Intermediate | 1-2 | Struct padding, alignment, tagged unions |
| 6 | Pattern Matching and Control Flow | Intermediate | 1 | Branch prediction, exhaustiveness |
| 7 | Error Handling the Systems Way | Intermediate | 1-2 | Error codes vs exceptions, Result/Option |
| 8 | Traits: Interfaces for Zero-Cost Abstraction | Intermediate | 2-3 | Vtables, static vs dynamic dispatch |
| 9 | Generics and Monomorphization | Intermediate | 1-2 | Code generation, binary size |
| 10 | Iterators and Closures | Intermediate | 1-2 | Inlining, zero-cost iteration |
| 11 | Modules, Crates, and the Build System | Intermediate | 1 | Linking, compilation units |
| 12 | Smart Pointers and Interior Mutability | Intermediate-Advanced | 2-3 | Reference counting, memory models |
| 13 | Collections Deep Dive | Intermediate-Advanced | 1-2 | Hash maps, B-trees, cache effects |
| 14 | File I/O and the Operating System | Intermediate-Advanced | 2-3 | File descriptors, syscalls, buffering |
| 15 | Concurrency: Threads and Shared State | Advanced | 2-3 | OS threads, mutexes, atomics |
| 16 | Concurrency: Message Passing and Async | Advanced | 2-3 | Channels, event loops, futures |
| 17 | Unsafe Rust and FFI | Advanced | 2-3 | Raw pointers, C ABI, UB |
| 18 | Lifetimes In Depth | Advanced | 2-3 | Dangling pointers, variance |
| 19 | Building a Real Networked Application | Advanced | 3-4 | TCP/IP, sockets, protocols |
| 20 | Performance, Profiling, and Optimization | Advanced | 2-3 | CPU caches, benchmarking, flamegraphs |

---

## Chapter 1: Bytes, Memory Layout, and Primitive Types

**Difficulty:** Beginner | **Sessions:** 1-2

### Learning Objectives
- Understand how data is represented as bytes in memory
- Know the size and layout of Rust's primitive types
- Use `std::mem::size_of` and `std::mem::align_of`
- Understand signed vs unsigned integers at the bit level
- Grasp why Rust requires explicit type conversions

### Systems Knowledge
> **How computers store data:** Everything in memory is bytes. An `i32` is 4 bytes interpreted as a two's complement signed integer. A `bool` is 1 byte even though it only needs 1 bit. A `char` is 4 bytes (Unicode scalar value), not 1 byte like in C. Understanding this is foundational -- every Rust rule about types traces back to how bytes are laid out in memory.

### Concepts
- Primitive types: `i8/i16/i32/i64/i128`, `u8/u16/u32/u64/u128`, `f32/f64`, `bool`, `char`
- Integer overflow behavior (debug vs release)
- Type casting with `as`
- Byte order (endianness)
- The `std::mem` module

### Exercises

**Exercise 1.1 -- Type Explorer**
Write a program that prints the size and alignment of every primitive type. Use `std::mem::size_of::<T>()` and `std::mem::align_of::<T>()`. Format it as a neat table.

**Exercise 1.2 -- Bit Patterns**
Write a function that takes a `u8` and prints its binary representation (without using format specifiers). Then write a function that converts an `i8` to its two's complement binary form. Verify that `-1i8` is `11111111` in binary.

**Exercise 1.3 -- Endianness Detector**
Write a program that determines whether your machine is little-endian or big-endian by creating a `u16` value and examining its bytes using `to_ne_bytes()`. Compare with `to_le_bytes()` and `to_be_bytes()`.

### Mini-Project: Hex Dump Tool
Build a command-line hex dump tool (like `xxd` or `hexdump`). It reads a file (passed as a command-line argument) and prints its contents in hex + ASCII format, 16 bytes per line, with offset addresses on the left. This teaches file I/O basics while reinforcing byte-level thinking.

Example output:
```
00000000: 4865 6c6c 6f2c 2077 6f72 6c64 210a 0000  Hello, world!...
```

---

## Chapter 2: The Stack: Functions, Ownership, and Scope

**Difficulty:** Beginner | **Sessions:** 1-2

### Learning Objectives
- Understand the call stack and stack frames
- See how local variables are allocated on the stack
- Understand ownership as stack-frame lifetime
- Learn move semantics for stack-allocated types
- Grasp `Copy` vs `Move` types and why the distinction exists

### Systems Knowledge
> **The call stack:** When a function is called, the CPU pushes a new "frame" onto the stack containing space for local variables and the return address. When the function returns, the frame is popped. This is why local variables are automatically cleaned up -- their memory is literally gone when the frame is popped. Rust's ownership model maps directly onto this: a value "owned" by a variable lives in that variable's stack frame. When the frame is gone, so is the value. This is RAII (Resource Acquisition Is Initialization), and it is the foundation of Rust's memory safety.

### Concepts
- Stack frames and how they grow/shrink
- Variable binding and shadowing
- Ownership rules (each value has exactly one owner)
- Move semantics (what happens to the bits)
- `Copy` trait and why primitives implement it
- `Drop` trait and RAII
- Block expressions and scope

### Exercises

**Exercise 2.1 -- Stack Visualizer**
Write a program with 3 nested function calls. In each function, declare a local variable and print its memory address using `&var as *const _ as usize`. Observe that stack addresses decrease (on most platforms). Draw the stack layout on paper.

**Exercise 2.2 -- Move Detective**
Write code that demonstrates move semantics: create a `String`, move it to another variable, and try to use the original (observe the compile error). Then do the same with an `i32` and observe that it copies instead. Write comments explaining WHY based on what you know about stack and heap.

**Exercise 2.3 -- Drop Order**
Create a struct `Noisy` that prints a message in its `Drop` implementation. Create several `Noisy` values in a function and predict the order they will be dropped. Verify by running the program. Then try putting them in a `Vec` and see what happens.

### Mini-Project: Expression Evaluator
Build a simple arithmetic expression evaluator. Parse strings like `"(2 + 3) * 4"` into a tree structure and evaluate them. This naturally exercises ownership because tree nodes own their children. Use an enum for the tree:
```rust
enum Expr {
    Num(f64),
    Add(Box<Expr>, Box<Expr>),
    Mul(Box<Expr>, Box<Expr>),
    // etc.
}
```
Start with just `+`, `-`, `*`, `/` and parentheses. The parser will exercise stack-based thinking (recursive descent is literally a stack of function calls).

---

## Chapter 3: The Heap: Boxes, Strings, and Vectors

**Difficulty:** Beginner | **Sessions:** 1-2

### Learning Objectives
- Understand heap allocation and why it exists
- Know the difference between `String` and `&str` at the memory level
- Understand `Vec<T>` internals (pointer, length, capacity)
- Use `Box<T>` for heap allocation
- Grasp when and why heap allocation happens

### Systems Knowledge
> **Stack vs Heap:** The stack is fast (just moving a pointer) but limited: you must know sizes at compile time, and data dies when the function returns. The heap is a large pool of memory managed by an allocator (like `malloc` in C). Allocation is slower (the allocator must find free space) but flexible: data lives until explicitly freed. In Ruby, almost everything is heap-allocated and garbage-collected. In Rust, you choose: stack when you can, heap when you must. `String` is a `(pointer, length, capacity)` triple on the stack pointing to heap-allocated bytes. Understanding this layout explains why `String` moves instead of copying.

### Concepts
- Heap allocation via `Box::new()`
- `String` vs `&str`: owned vs borrowed, heap vs stack/static
- `Vec<T>` growth strategy (capacity doubling)
- `String` is really a `Vec<u8>` with UTF-8 guarantee
- Deref coercion: why `&String` works as `&str`
- The allocator: what happens when you call `Vec::push()`

### Exercises

**Exercise 3.1 -- String Anatomy**
Write a program that creates a `String`, then prints its pointer (`as_ptr()`), length (`len()`), and capacity (`capacity()`). Push characters one at a time and print these values after each push. Observe when reallocation happens.

**Exercise 3.2 -- Box Layout**
Create a `Box<i32>` and print the address of the Box itself (on the stack) and the address of the data it points to (on the heap). Compare the addresses -- the stack address should be high, the heap address low (on typical systems).

**Exercise 3.3 -- Vec Internals**
Implement a simplified `MyVec<i32>` struct that stores a pointer, length, and capacity. Implement `new()`, `push()`, `pop()`, and `get()`. Handle reallocation when capacity is exceeded. This will require `unsafe` for raw pointer operations -- that is fine, the point is to understand what `Vec` does internally.

### Mini-Project: Text Statistics Tool
Build a CLI tool that reads a text file and reports statistics: word count, line count, character count, most frequent words (top 10), average word length, and longest line. Use `String`, `Vec`, and `HashMap`. This reinforces heap-allocated collections and string processing. Compare your results with `wc` for the simple counts.

---

## Chapter 4: References, Borrowing, and the Borrow Checker

**Difficulty:** Beginner-Intermediate | **Sessions:** 2-3

### Learning Objectives
- Understand references as pointers with compile-time guarantees
- Master the borrowing rules (one mutable XOR many immutable)
- Understand why these rules prevent data races and aliasing bugs
- Work with slices as fat pointers
- Understand lifetime annotations at a basic level

### Systems Knowledge
> **Pointer aliasing and its dangers:** In C, you can have two pointers to the same data and mutate through either one. This causes bugs: iterator invalidation (modifying a collection while iterating), data races (two threads writing simultaneously), and use-after-free (accessing memory through a stale pointer). Rust's borrowing rules prevent ALL of these at compile time. The rule is simple: at any point, you can have either ONE mutable reference OR any number of immutable references. This is not arbitrary -- it directly prevents the aliasing + mutation combination that causes these bugs. Slices (`&[T]`) are "fat pointers": a pointer plus a length, giving bounds-checked access to contiguous data.

### Concepts
- References (`&T` and `&mut T`) are pointers
- Borrowing rules and why they exist
- Reborrowing
- Slices: `&[T]`, `&str`
- The "aliasing XOR mutation" principle
- Iterator invalidation (impossible in safe Rust)
- Basic lifetime annotations (`'a`)

### Exercises

**Exercise 4.1 -- Reference Addresses**
Write a program that creates a value, takes a reference to it, and prints both the value's address and the reference's own address (a reference is itself stored somewhere). Demonstrate that `&T` is literally a memory address by casting it to `*const T` and printing it.

**Exercise 4.2 -- Borrow Checker Battles**
Write 5 snippets of code that violate borrowing rules in different ways. For each one, write a comment explaining what real-world bug the borrow checker is preventing (e.g., iterator invalidation, data race, use-after-free). Then fix each snippet.

**Exercise 4.3 -- Slice Operations**
Implement these functions using slices (not Vec):
- `fn longest_word(s: &str) -> &str` -- returns the longest word
- `fn second_largest(nums: &[i32]) -> Option<i32>` -- returns the second largest element
- `fn is_sorted(nums: &[i32]) -> bool` -- checks if a slice is sorted

### Mini-Project: CSV Parser Library
Build a simple CSV parser that reads CSV data and provides access to rows and columns. The key challenge: design the API around references and lifetimes. The parser should borrow the input string rather than copying data. Implement:
- `CsvParser::new(data: &str) -> CsvParser` (borrows the data)
- `fn headers(&self) -> Vec<&str>`
- `fn rows(&self) -> Vec<Vec<&str>>`
- `fn get_column(&self, name: &str) -> Vec<&str>`

This is a natural exercise in lifetime management because all the parsed data references the original string.

---

## Chapter 5: Structs, Enums, and Data Layout

**Difficulty:** Intermediate | **Sessions:** 1-2

### Learning Objectives
- Design custom types with structs and enums
- Understand struct padding and alignment in memory
- See how Rust enums are tagged unions
- Understand the niche optimization (e.g., `Option<Box<T>>` is pointer-sized)
- Implement methods with `impl` blocks

### Systems Knowledge
> **Struct padding and alignment:** CPUs access memory most efficiently when data is aligned to its size boundary (a `u32` at an address divisible by 4). Compilers insert "padding" bytes between struct fields to maintain alignment. This means field order affects struct size! Rust may reorder fields for optimal layout (unlike C). Enums are "tagged unions": a tag byte (the discriminant) plus enough space for the largest variant. Rust optimizes this brilliantly: `Option<&T>` uses the null pointer as `None`, so it is the same size as a raw pointer. Understanding this layout helps you write cache-friendly, memory-efficient code.

### Concepts
- Named structs, tuple structs, unit structs
- `#[repr(C)]` vs default Rust layout
- Enum discriminants and tagged unions
- Option/Result as enums
- Niche optimization
- Method syntax and `self`/`&self`/`&mut self`
- Associated functions

### Exercises

**Exercise 5.1 -- Layout Explorer**
Create several structs with fields in different orders and use `std::mem::size_of` to measure their sizes. Find a case where reordering fields changes the size. Use `#[repr(C)]` and compare.

**Exercise 5.2 -- Niche Optimization**
Print the size of `Option<Box<i32>>`, `Option<&i32>`, `Option<bool>`, `Option<i32>`, and `Option<Option<bool>>`. Explain why each is the size it is.

**Exercise 5.3 -- State Machine**
Model a TCP connection state machine using an enum where each variant holds different data:
```rust
enum TcpState {
    Closed,
    Listen { port: u16 },
    SynSent { seq: u32 },
    Established { seq: u32, ack: u32, window: u16 },
    // etc.
}
```
Implement transition methods that enforce valid state transitions at the type level.

### Mini-Project: Entity Component System (ECS) Prototype
Build a tiny ECS (used in game engines). Define components as structs (Position, Velocity, Health, etc.), entities as IDs, and a World that stores components in contiguous arrays. Implement a simple system that updates positions based on velocities. This teaches struct layout, cache-friendly data design, and the "data-oriented" mindset that Rust encourages.

---

## Chapter 6: Pattern Matching and Control Flow

**Difficulty:** Intermediate | **Sessions:** 1

### Learning Objectives
- Master `match`, `if let`, `while let`
- Understand exhaustiveness checking
- Use pattern matching with destructuring
- Apply pattern matching to complex data structures

### Systems Knowledge
> **Branch prediction and pattern matching:** Modern CPUs predict which branch of an `if` statement will be taken. Mispredictions are expensive (~15 CPU cycles wasted). Rust's `match` compiles to efficient jump tables or decision trees, which can be more predictable than chains of `if/else`. Exhaustiveness checking is a compile-time guarantee that you handle all cases -- this prevents an entire class of bugs where a new enum variant is added but some code forgets to handle it. In C, a missed `switch` case is a silent bug. In Rust, it is a compile error.

### Concepts
- `match` expressions and exhaustiveness
- Destructuring structs, enums, tuples
- Match guards
- `if let` and `while let`
- Nested patterns
- The `@` binding operator
- Or-patterns with `|`

### Exercises

**Exercise 6.1 -- Command Parser**
Write a function that takes a string command (like `"move north 3"`, `"attack goblin"`, `"look"`) and uses pattern matching on the split words to return a `Command` enum. Handle all edge cases.

**Exercise 6.2 -- Deep Destructuring**
Given a nested data structure (e.g., `Vec<Result<(String, Vec<i32>), Error>>`), write match expressions that extract specific data at various nesting levels.

**Exercise 6.3 -- Expression Simplifier**
Using the `Expr` enum from Chapter 2, implement an `optimize()` function that simplifies expressions: `x * 1 => x`, `x + 0 => x`, `0 * x => 0`, etc. Use nested pattern matching.

### Mini-Project: JSON Pretty Printer
Define a `JsonValue` enum and implement a parser for a subset of JSON (strings, numbers, booleans, null, arrays, objects). Then implement a pretty printer that formats the JSON with proper indentation. Pattern matching is the natural tool for both parsing and formatting recursive data structures.

---

## Chapter 7: Error Handling the Systems Way

**Difficulty:** Intermediate | **Sessions:** 1-2

### Learning Objectives
- Understand the trade-offs between exceptions, error codes, and Result types
- Use `Result<T, E>` and `Option<T>` idiomatically
- Master the `?` operator
- Design custom error types
- Use the `thiserror` and `anyhow` crates

### Systems Knowledge
> **Error handling across languages:** Ruby uses exceptions (stack unwinding -- expensive, invisible control flow). C uses error codes (cheap but easy to ignore). Go uses multiple return values (explicit but verbose). Rust's `Result<T, E>` is a zero-cost abstraction: it is just an enum (tagged union in memory), the `?` operator compiles to a simple branch, and the compiler forces you to handle errors. No hidden control flow, no forgotten error checks, no runtime cost beyond the branch. Panics exist for truly unrecoverable situations and do unwind the stack (or abort), similar to exceptions but not meant for normal error handling.

### Concepts
- `Result<T, E>` and `Option<T>` combinators
- The `?` operator and early return
- Custom error types with `enum`
- `From` trait for error conversion
- `thiserror` for library errors
- `anyhow` for application errors
- Panic vs Result: when to use which
- `unwrap()` and when it is acceptable

### Exercises

**Exercise 7.1 -- Error Conversion Chain**
Write a function that reads a config file, parses it as integer values, and returns their sum. Chain three different error types (IO error, parse error, custom validation error) using `From` implementations and `?`.

**Exercise 7.2 -- Custom Error Type**
Design an error enum for a calculator that handles division by zero, overflow, invalid input, and unknown functions. Implement `std::fmt::Display` and `std::error::Error` for it, both manually and with `thiserror`.

**Exercise 7.3 -- Option Gymnastics**
Write a function that navigates a nested `HashMap<String, HashMap<String, Vec<i32>>>` to find a specific value, using only `Option` combinators (`and_then`, `map`, `unwrap_or`, etc.) -- no explicit `match` or `if let`.

### Mini-Project: Config File Parser
Build a config file parser (INI or TOML subset) with rich error reporting. Errors should include the file name, line number, column, and a helpful message. Implement both a library version (with typed errors using `thiserror`) and a CLI wrapper (using `anyhow`). This teaches the library vs application error handling split that is common in Rust.

---

## Chapter 8: Traits: Interfaces for Zero-Cost Abstraction

**Difficulty:** Intermediate | **Sessions:** 2-3

### Learning Objectives
- Define and implement traits
- Understand static dispatch vs dynamic dispatch
- Know when to use `impl Trait` vs `dyn Trait`
- Understand vtables and their memory layout
- Use common standard library traits

### Systems Knowledge
> **Static vs Dynamic Dispatch:** When you call a method through a trait, the compiler has two choices. Static dispatch (using `impl Trait` or generics) resolves the concrete type at compile time -- the compiler generates specialized code for each type, which can then be inlined. Zero runtime cost. Dynamic dispatch (using `dyn Trait`) uses a vtable: a table of function pointers stored alongside the data pointer in a "fat pointer" (16 bytes on 64-bit systems). The CPU must follow the pointer to find the function, which prevents inlining and can cause cache misses. Choose static dispatch by default; use dynamic dispatch when you need type erasure or have many types.

### Concepts
- Trait definition and implementation
- Default methods
- Trait bounds
- `impl Trait` in argument and return position
- `dyn Trait` and trait objects
- Vtable layout
- Supertraits
- Common traits: `Display`, `Debug`, `Clone`, `Default`, `PartialEq`, `Eq`, `Hash`, `PartialOrd`, `Ord`
- Operator overloading traits
- `Deref` and `DerefMut`

### Exercises

**Exercise 8.1 -- Shape Hierarchy**
Define a `Shape` trait with `area()` and `perimeter()`. Implement it for `Circle`, `Rectangle`, and `Triangle`. Write a function using static dispatch (`impl Shape`) and another using dynamic dispatch (`dyn Shape`). Print the sizes of references to each.

**Exercise 8.2 -- Vtable Explorer**
Use `std::mem::size_of` to compare the size of `&T`, `&dyn Trait`, and `Box<dyn Trait>`. Write a function that takes a `&dyn Shape` and uses unsafe code to print the raw vtable pointer. (Educational purpose only.)

**Exercise 8.3 -- Trait Composition**
Define traits `Readable`, `Writable`, and `Seekable`. Combine them with supertraits to create `ReadWrite` and `ReadWriteSeek`. Implement these for a `Buffer` struct. This mirrors how `std::io` traits work.

### Mini-Project: Plugin System
Build a simple plugin system for a text processor. Define a `Plugin` trait with methods like `name()`, `process(text: &str) -> String`. Implement several plugins (uppercase, word count, reverse, rot13). Build a registry that stores `Box<dyn Plugin>` and can apply plugins by name. Add a pipeline feature that chains plugins. This teaches trait objects, dynamic dispatch, and the strategy pattern.

---

## Chapter 9: Generics and Monomorphization

**Difficulty:** Intermediate | **Sessions:** 1-2

### Learning Objectives
- Write generic functions and types
- Understand monomorphization and its trade-offs
- Use multiple trait bounds and `where` clauses
- Understand turbofish syntax and type inference
- Grasp the impact on compile time and binary size

### Systems Knowledge
> **Monomorphization:** When you write `fn foo<T: Display>(x: T)`, the compiler generates a separate, specialized copy of `foo` for every concrete type it is called with. `foo::<i32>` and `foo::<String>` become two different functions in the binary. This is why generics are "zero-cost" -- each specialized version can be fully optimized with inlining, constant folding, etc. The trade-off: more copies means larger binaries and longer compile times. This is the opposite of Java/C# generics (which use type erasure/boxing at runtime). Understanding this helps you make informed decisions about generics vs trait objects.

### Concepts
- Generic functions, structs, enums, impls
- Trait bounds: `T: Trait`, `T: Trait1 + Trait2`
- `where` clauses
- Turbofish `::<>`
- Const generics
- Phantom types
- Monomorphization and binary size
- When to use generics vs `dyn Trait`

### Exercises

**Exercise 9.1 -- Generic Data Structures**
Implement a generic `Stack<T>` with `push`, `pop`, `peek`, and `is_empty`. Then add trait bounds to implement `Display` for `Stack<T>` only when `T: Display`.

**Exercise 9.2 -- Type-State Pattern**
Implement a builder for an HTTP request using the type-state pattern. The builder should enforce at compile time that a URL is set before building. Use phantom types to track the state.

**Exercise 9.3 -- Monomorphization Cost**
Write a generic function and call it with 5 different types. Use `cargo build --release` and examine the binary size. Then rewrite it using `dyn Trait` and compare. Use `nm` or `objdump` to count the generated function copies.

### Mini-Project: Generic Sorted Collection
Implement a generic sorted collection (`SortedVec<T: Ord>`) that maintains elements in sorted order. Implement `insert`, `contains` (using binary search), `remove`, `iter`, `merge` (merge two sorted collections), and `from_iter`. Implement standard traits: `FromIterator`, `IntoIterator`, `Index`, `Display`, `Debug`, `Clone`. Write comprehensive tests. This is a thorough exercise in generics and trait implementations.

---

## Chapter 10: Iterators and Closures

**Difficulty:** Intermediate | **Sessions:** 1-2

### Learning Objectives
- Understand the `Iterator` trait and lazy evaluation
- Write custom iterators
- Use closures and understand their capture semantics
- Know how closures are represented in memory (Fn, FnMut, FnOnce)
- Chain iterator adaptors for data processing

### Systems Knowledge
> **Zero-cost iteration:** In C, a `for` loop with an index is the baseline. In Ruby, iterators like `.map.select.reduce` create intermediate arrays at each step. In Rust, iterator chains like `.iter().filter().map().collect()` compile down to a single loop with no intermediate allocations -- the compiler fuses the entire chain. This is because each adaptor is a struct holding the previous iterator, and the `next()` calls inline into a tight loop. Closures are also zero-cost: a closure that captures `&x` is literally a struct containing `&x`, and calls to it inline completely. The three closure traits (`Fn`, `FnMut`, `FnOnce`) correspond to `&self`, `&mut self`, and `self` -- they map to how the closure's captured state is accessed.

### Concepts
- `Iterator` trait and `next()`
- Iterator adaptors: `map`, `filter`, `take`, `skip`, `chain`, `zip`, `enumerate`, `flat_map`
- Consumers: `collect`, `fold`, `sum`, `any`, `all`, `find`, `count`
- Custom iterators with `impl Iterator`
- Closures and capture modes (by reference, mutable reference, move)
- `Fn`, `FnMut`, `FnOnce`
- `impl Fn` vs `dyn Fn`

### Exercises

**Exercise 10.1 -- Iterator Chain Challenge**
Solve these using only iterator chains (no explicit loops):
- Find the sum of squares of all odd numbers in a Vec
- Flatten a `Vec<Vec<i32>>` and find the 3 largest values
- Zip two vectors and find pairs where the sum exceeds a threshold

**Exercise 10.2 -- Custom Iterator**
Implement a `Fibonacci` iterator that yields Fibonacci numbers. Then implement a `Windows` iterator that yields overlapping windows of a given size from a slice (like `slice.windows()` but built from scratch).

**Exercise 10.3 -- Closure Investigation**
Write closures that capture a variable by shared reference, mutable reference, and by move. Print the size of each closure using `std::mem::size_of_val`. Explain the sizes based on what is captured.

### Mini-Project: Data Pipeline Tool
Build a CLI tool that reads CSV data from stdin and supports a pipeline of transformations specified as command-line arguments: `filter`, `map`, `sort`, `unique`, `head`, `tail`, `count`, `sum`, `avg`. For example: `cat data.csv | pipeline filter "age > 30" | pipeline sort "name" | pipeline head 10`. Each transformation is implemented as an iterator adaptor. This teaches composable, zero-cost data processing.

---

## Chapter 11: Modules, Crates, and the Build System

**Difficulty:** Intermediate | **Sessions:** 1

### Learning Objectives
- Organize code with modules and visibility
- Understand the crate system and Cargo
- Use external crates from crates.io
- Understand feature flags and conditional compilation
- Know how Rust compiles and links code

### Systems Knowledge
> **Compilation and linking:** In Ruby, you `require` a file and the interpreter loads it. In Rust, the compiler processes one crate at a time, producing an object file (machine code). The linker then combines object files into a final binary. `cargo build` orchestrates this, compiling dependencies in parallel. Static linking bundles everything into one binary (Rust's default). Dynamic linking shares `.so`/`.dylib` files at runtime. Understanding this explains why Rust binaries are large but self-contained, why compile times scale with crate count, and why splitting code into crates can improve incremental compilation.

### Concepts
- `mod`, `pub`, `use`
- File structure: `mod.rs` vs `folder/mod.rs` vs `folder.rs`
- Visibility rules: `pub`, `pub(crate)`, `pub(super)`
- `Cargo.toml` and dependencies
- Workspaces
- Feature flags
- `#[cfg()]` conditional compilation
- Build scripts (`build.rs`)
- Documentation with `///` and `cargo doc`

### Exercises

**Exercise 11.1 -- Library Structure**
Take the CSV parser from Chapter 4 and restructure it as a proper library crate with: a public API module, an internal parser module, an error module, and integration tests. Use appropriate visibility modifiers.

**Exercise 11.2 -- Feature Flags**
Add a feature flag to your CSV parser: `serde` support. When the `serde` feature is enabled, derive `Serialize`/`Deserialize` for your types. Use `#[cfg(feature = "serde")]`.

**Exercise 11.3 -- Workspace**
Create a Cargo workspace with: a library crate (the CSV parser), a CLI binary crate that uses it, and a benchmarks crate. Configure shared dependencies.

### Mini-Project: Cargo Subcommand
Build a simple `cargo` subcommand (e.g., `cargo-linecount` that counts lines of Rust code in a project, excluding tests and comments). This teaches how the Rust ecosystem tooling works, and requires reading `Cargo.toml`, traversing project structure, and producing formatted output.

---

## Chapter 12: Smart Pointers and Interior Mutability

**Difficulty:** Intermediate-Advanced | **Sessions:** 2-3

### Learning Objectives
- Understand `Rc<T>`, `Arc<T>`, and reference counting
- Use `RefCell<T>` and understand interior mutability
- Know when to use `Cell<T>` vs `RefCell<T>`
- Understand `Cow<T>` (clone-on-write)
- See how these relate to garbage collection

### Systems Knowledge
> **Reference counting vs garbage collection:** Ruby uses a combination of reference counting and mark-and-sweep GC. The GC periodically stops the world to trace live objects. Rust's `Rc<T>` is pure reference counting: a counter alongside the data tracks how many `Rc` pointers exist. When the count hits zero, the value is dropped immediately (deterministic destruction). `Arc<T>` is the same but uses atomic operations for thread safety (more expensive due to CPU cache coherence protocols). `RefCell<T>` moves borrow checking to runtime -- it maintains a counter of active borrows and panics if rules are violated. This is Rust's escape hatch for when the compile-time borrow checker is too conservative, but it is still safe (panics instead of UB).

### Concepts
- `Rc<T>`: shared ownership, reference counting
- `Arc<T>`: atomic reference counting for threads
- `RefCell<T>`: runtime borrow checking
- `Cell<T>`: copy-based interior mutability
- `Cow<T>`: clone-on-write optimization
- Weak references and preventing cycles
- Comparison with GC (Ruby's approach)

### Exercises

**Exercise 12.1 -- Shared Ownership Graph**
Build a simple graph (nodes connected by edges) using `Rc<RefCell<Node>>`. Implement adding nodes, connecting them, and traversing. Observe what happens with cycles (memory leak). Fix it with `Weak<T>`.

**Exercise 12.2 -- Interior Mutability Patterns**
Implement a caching wrapper: `CachedComputation<T>` that takes a closure and only calls it once, caching the result. Use `RefCell` (or `Cell` if the value is `Copy`). Then implement a thread-safe version with `Mutex`.

**Exercise 12.3 -- Cow Optimization**
Write a function `fn process(input: &str) -> Cow<str>` that returns the input unchanged if it is already valid, or a modified copy if it needs fixing (e.g., normalizing whitespace). Benchmark to show that `Cow` avoids allocation in the common case.

### Mini-Project: Doubly-Linked List
Implement a doubly-linked list using `Rc<RefCell<Node>>` and `Weak`. Implement `push_front`, `push_back`, `pop_front`, `pop_back`, `iter` (forward and backward), and `Display`. This is famously tricky in Rust and teaches why Rust makes shared mutable state difficult -- because it IS difficult to get right. Include a reflection on why a `Vec` is almost always better in practice (cache locality, simpler code).

---

## Chapter 13: Collections Deep Dive

**Difficulty:** Intermediate-Advanced | **Sessions:** 1-2

### Learning Objectives
- Understand the internal implementation of HashMap, BTreeMap, HashSet, BTreeSet, VecDeque, BinaryHeap
- Know the performance characteristics and when to use each
- Understand hashing, hash collisions, and their impact
- Design cache-friendly data structures

### Systems Knowledge
> **Cache effects on data structure performance:** Modern CPUs have a hierarchy of caches (L1: ~4 cycles, L2: ~12 cycles, L3: ~40 cycles, main memory: ~200 cycles). `Vec<T>` stores elements contiguously, so iterating is cache-friendly (prefetcher loads the next cache line). `LinkedList` scatters nodes across the heap, causing cache misses on every access. This is why `Vec` beats `LinkedList` for almost everything in practice, despite Big-O suggesting otherwise. `HashMap` uses "Swiss Table" (hashbrown): it stores metadata in a separate array for SIMD-accelerated probing, making lookups fast even at high load factors. `BTreeMap` stores multiple keys per node for cache efficiency.

### Concepts
- `HashMap` / `HashSet`: hashing, SwissTable, load factor
- `BTreeMap` / `BTreeSet`: B-tree structure, sorted operations
- `VecDeque`: ring buffer
- `BinaryHeap`: max-heap in an array
- The `Entry` API for efficient map operations
- Custom `Hash` implementations
- Cache-friendly vs cache-hostile layouts

### Exercises

**Exercise 13.1 -- Hash Collision Experiment**
Implement a pathological `Hash` that always returns the same value. Benchmark `HashMap` operations with this hash vs the default. Observe the dramatic performance difference and explain why.

**Exercise 13.2 -- The Right Collection**
For each scenario, choose the optimal collection and justify your choice: frequent lookups by key, maintaining sorted order, FIFO queue, priority queue, deduplication while preserving insertion order.

**Exercise 13.3 -- Entry API Mastery**
Write a word frequency counter three ways: with `contains_key` + `insert`, with `or_insert`, and with `entry().and_modify().or_insert()`. Benchmark all three. Explain why the Entry API is faster (hint: single hash lookup).

### Mini-Project: LRU Cache
Implement an LRU (Least Recently Used) cache with O(1) get and put operations. Use a `HashMap` for O(1) lookup and a doubly-linked list (or `VecDeque`) for O(1) eviction. Support a configurable capacity. Write benchmarks comparing your LRU cache performance with different backing stores. Add an optional TTL (time-to-live) feature.

---

## Chapter 14: File I/O and the Operating System

**Difficulty:** Intermediate-Advanced | **Sessions:** 2-3

### Learning Objectives
- Understand file descriptors and how the OS manages files
- Use `std::fs` and `std::io` for file operations
- Implement buffered I/O and understand why it matters
- Work with paths, directories, and file metadata
- Understand memory-mapped files

### Systems Knowledge
> **File descriptors and system calls:** When you open a file, the OS kernel creates an entry in a per-process table and returns a small integer: the file descriptor (fd). Every read/write goes through a system call (a controlled transition from user space to kernel space), which is expensive (~100ns). This is why buffered I/O exists: `BufReader` accumulates reads into a large buffer (default 8KB), making fewer syscalls. In Ruby, IO is always buffered. In Rust, `File` is unbuffered by default -- every `read` call is a syscall. Wrapping it in `BufReader` can be 10-100x faster for small reads. Memory-mapped files (`mmap`) map a file directly into your process's address space, letting you access file contents as if they were in memory.

### Concepts
- `File`, `BufReader`, `BufWriter`
- `Read`, `Write`, `Seek` traits
- File descriptors under the hood
- `Path` and `PathBuf`
- Directory traversal with `std::fs::read_dir`
- Temporary files
- File permissions and metadata
- Memory-mapped files with `memmap2` crate
- stdin, stdout, stderr as file descriptors (0, 1, 2)

### Exercises

**Exercise 14.1 -- Buffering Benchmark**
Write a program that reads a large file byte by byte using: (a) unbuffered `File`, (b) `BufReader` with default buffer, (c) `BufReader` with 64KB buffer. Time each approach. The difference should be dramatic.

**Exercise 14.2 -- File Descriptor Explorer**
Write a program that opens several files, then reads `/dev/fd/` (on macOS) or `/proc/self/fd/` (on Linux) to list all open file descriptors. Print the fd number and what file it points to.

**Exercise 14.3 -- Recursive Directory Walker**
Implement a recursive directory walker that finds files matching a pattern (like a simple `find` command). Use `std::fs::read_dir` and handle errors gracefully (permission denied, symlink loops). Support basic glob patterns.

### Mini-Project: File Synchronization Tool
Build a one-way file sync tool (like a simplified `rsync`). Given a source and destination directory, it should: compare files by size and modification time, copy new and modified files, optionally delete files in destination that do not exist in source, display a dry-run summary before executing, and show progress during copy. Use buffered I/O for efficient copying. This combines file I/O, path manipulation, error handling, and CLI design.

---

## Chapter 15: Concurrency: Threads and Shared State

**Difficulty:** Advanced | **Sessions:** 2-3

### Learning Objectives
- Understand OS threads and how they work
- Use `std::thread` and `JoinHandle`
- Share data between threads safely with `Mutex` and `RwLock`
- Understand `Send` and `Sync` traits
- Know what atomics are and when to use them

### Systems Knowledge
> **OS threads and synchronization:** An OS thread has its own stack but shares the process's heap memory. The OS scheduler switches between threads (context switch: ~1-10 microseconds). Sharing mutable data between threads is dangerous: two threads writing to the same memory simultaneously causes a data race (undefined behavior in C/C++). A Mutex (mutual exclusion) is a lock: only one thread can hold it at a time. Acquiring a lock requires an atomic CPU instruction (like compare-and-swap) and possibly a syscall if the lock is contended. `RwLock` allows multiple readers OR one writer. Rust makes data races a compile-time error through `Send` (can be transferred to another thread) and `Sync` (can be shared between threads). `Rc` is not `Send` because its reference count is not atomic. `Arc` is `Send` because it uses atomic operations.

### Concepts
- Creating threads with `std::thread::spawn`
- `JoinHandle` and waiting for threads
- `Mutex<T>` and `MutexGuard` (RAII locking)
- `RwLock<T>`
- `Arc<Mutex<T>>` pattern
- `Send` and `Sync` marker traits
- Thread-local storage
- Atomics: `AtomicBool`, `AtomicUsize`, ordering
- Deadlock and how to avoid it
- Scoped threads (`std::thread::scope`)

### Exercises

**Exercise 15.1 -- Parallel Sum**
Split a large vector into chunks, sum each chunk in a separate thread, then combine the results. Compare performance with the single-threaded version. Try with 1, 2, 4, 8 threads and measure the speedup.

**Exercise 15.2 -- Thread-Safe Counter**
Implement a thread-safe counter three ways: with `Mutex<i64>`, with `AtomicI64`, and with message passing (channel). Benchmark all three under contention (many threads incrementing simultaneously). Explain the performance differences.

**Exercise 15.3 -- Deadlock Demonstration**
Write code that deadlocks (two mutexes locked in different orders by different threads). Then fix it using consistent lock ordering. Write comments explaining why deadlock occurred and how the fix works.

### Mini-Project: Parallel File Processor
Build a multi-threaded file processor: given a directory of text files, count word frequencies across all files using a thread pool. Each worker thread processes one file, then merges its results into a shared `HashMap`. Compare: (a) single `Arc<Mutex<HashMap>>`, (b) per-thread local maps merged at the end, (c) using a channel to send results. Measure and compare performance. This teaches real-world concurrent data processing.

---

## Chapter 16: Concurrency: Message Passing and Async

**Difficulty:** Advanced | **Sessions:** 2-3

### Learning Objectives
- Use channels for message passing between threads
- Understand the async/await model
- Use the `tokio` runtime
- Know the difference between threads and async tasks
- Build async I/O applications

### Systems Knowledge
> **Event loops and async I/O:** Threads work but are expensive: each thread uses ~8KB-8MB of stack, and context switches add overhead. For I/O-heavy workloads (web servers handling thousands of connections), async I/O is more efficient. Instead of blocking a thread waiting for data, you register interest with the OS (`epoll` on Linux, `kqueue` on macOS) and get notified when data is ready. An event loop manages this, multiplexing many tasks onto a few threads. Rust's `async/await` compiles into state machines -- each `.await` point is a state transition. The `Future` trait is like `Iterator` but for async: `poll()` returns `Ready(value)` or `Pending`. Unlike JavaScript (single-threaded async), Rust's async is multi-threaded: tokio runs a thread pool of workers stealing tasks from each other.

### Concepts
- `mpsc` channels: `Sender` and `Receiver`
- `sync_channel` for bounded channels
- `async fn`, `.await`, `Future` trait
- `tokio` runtime: `#[tokio::main]`, `spawn`, `select!`
- `async` channels: `tokio::sync::mpsc`
- `Stream` trait (async iterator)
- Pinning and why it exists
- CPU-bound vs I/O-bound workloads
- When to use threads vs async

### Exercises

**Exercise 16.1 -- Channel Patterns**
Implement a producer-consumer system: multiple producer threads generate work items, a single consumer processes them. Use both unbounded and bounded channels. Observe what happens when the consumer is slower than the producers with a bounded channel.

**Exercise 16.2 -- Async Basics**
Write an async function that concurrently fetches multiple URLs (use `reqwest`). Compare sequential fetching vs concurrent (`tokio::join!`). Measure the wall-clock time difference.

**Exercise 16.3 -- Select**
Use `tokio::select!` to implement a timeout: race an async operation against a `tokio::time::sleep`. Also implement a simple actor that processes messages from a channel but shuts down when it receives a shutdown signal on a separate channel.

### Mini-Project: Chat Server
Build a multi-client chat server using `tokio`. Clients connect via TCP, send messages, and see messages from other clients. Features: named users, private messages, channel/room support, a `/list` command to see connected users. Use `tokio::sync::broadcast` for the message bus. This teaches real async I/O, protocol design, and concurrent state management.

---

## Chapter 17: Unsafe Rust and FFI

**Difficulty:** Advanced | **Sessions:** 2-3

### Learning Objectives
- Understand what `unsafe` means and does not mean
- Know the five things `unsafe` allows
- Write safe abstractions around unsafe code
- Call C functions from Rust (FFI)
- Expose Rust functions to C

### Systems Knowledge
> **The unsafe boundary:** Safe Rust guarantees no undefined behavior (UB). `unsafe` does not turn off the borrow checker -- it unlocks five specific powers: dereferencing raw pointers, calling unsafe functions, accessing mutable statics, implementing unsafe traits, and accessing union fields. The key insight: `unsafe` is not "dangerous code" but rather "code where the programmer vouches for invariants the compiler cannot check." Good Rust code uses `unsafe` internally to build safe abstractions (e.g., `Vec` uses raw pointers internally but exposes a safe API). FFI (Foreign Function Interface) is inherently unsafe because the compiler cannot verify C code's behavior. Understanding the unsafe boundary is essential for interoperating with the operating system, C libraries, and hardware.

### Concepts
- The five `unsafe` superpowers
- Raw pointers: `*const T` and `*mut T`
- `unsafe fn` vs `unsafe` block
- The `#[no_mangle]` and `extern "C"` attributes
- Calling C from Rust
- Calling Rust from C
- The `bindgen` tool
- Common UB: null pointer dereference, data races, invalid references
- Writing safe wrappers around unsafe code

### Exercises

**Exercise 17.1 -- Raw Pointer Arithmetic**
Implement a function that takes a `&[i32]` and uses raw pointer arithmetic to find the sum (no indexing, no iterators). This teaches how pointers and arrays relate at the machine level.

**Exercise 17.2 -- Safe Wrapper**
Write an unsafe function that splits a mutable slice into two mutable slices at a given index (reimplementing `split_at_mut`). Then wrap it in a safe function that validates the index. Explain why the safe version could not be written without `unsafe`.

**Exercise 17.3 -- C Interop**
Call the C `libc` functions `getpid()`, `gethostname()`, and `getcwd()` from Rust. Handle the C-style error reporting (return codes, errno) and convert results to Rust types.

### Mini-Project: SQLite Bindings
Build minimal Rust bindings for SQLite using FFI. Wrap the C API (`sqlite3_open`, `sqlite3_exec`, `sqlite3_prepare_v2`, `sqlite3_step`, `sqlite3_column_*`, `sqlite3_close`) in a safe Rust API. Create structs `Database` and `Statement` that manage lifetimes correctly (the statement must not outlive the database). Write tests that create a database, create a table, insert rows, and query them.

---

## Chapter 18: Lifetimes In Depth

**Difficulty:** Advanced | **Sessions:** 2-3

### Learning Objectives
- Understand lifetimes as scopes, not annotations
- Master lifetime elision rules
- Use multiple lifetime parameters
- Understand lifetime bounds on traits and generics
- Know about variance (covariance, invariance, contravariance)

### Systems Knowledge
> **Dangling pointers and lifetime analysis:** In C, a function can return a pointer to a local variable -- accessing it after the function returns is use-after-free (the memory is gone, you read garbage or crash). Lifetimes are Rust's compile-time solution: the annotation `'a` does not change runtime behavior -- it tells the compiler "this reference must not outlive scope `'a`." The compiler then verifies this statically. Lifetime elision rules let you omit annotations in common cases (the compiler infers them). Variance describes how lifetimes interact with generics: `&'a T` is covariant in `'a` (you can use a longer lifetime where a shorter one is expected), but `&'a mut T` is invariant in `T` (you cannot substitute a subtype). This prevents subtle bugs with mutable references.

### Concepts
- Lifetime annotations: `'a`, `'b`, `'static`
- Lifetime elision rules (3 rules)
- Multiple lifetimes and how they interact
- Lifetime bounds: `T: 'a`
- Higher-ranked trait bounds: `for<'a>`
- Variance: covariance, invariance, contravariance
- Self-referential structs (and why they are hard)
- The `'static` lifetime and `'static` bound
- Subtyping: `'long: 'short`

### Exercises

**Exercise 18.1 -- Lifetime Puzzles**
Given 5 function signatures with lifetime annotations, predict whether they compile and why. Then verify by running the compiler. Examples: returning a reference to a local, multiple references with different lifetimes, struct containing references.

**Exercise 18.2 -- Multi-Lifetime Struct**
Implement a `Merger<'a, 'b>` struct that holds references to two slices with independent lifetimes and produces a merged iterator. Explain why two separate lifetimes are needed.

**Exercise 18.3 -- Self-Referential Attempt**
Try to create a struct that contains a `String` and a `&str` pointing into that String. Observe why this fails. Then solve it using: (a) indices instead of references, (b) `Pin` + unsafe, (c) the `ouroboros` crate.

### Mini-Project: Streaming Parser
Build a streaming parser for a line-based protocol (e.g., HTTP headers or a log format). The parser borrows from an input buffer and yields parsed items that reference the buffer data. Handle the case where the buffer needs to be refilled (partial reads). This requires careful lifetime management because parsed items must not outlive the buffer contents they reference.

---

## Chapter 19: Building a Real Networked Application

**Difficulty:** Advanced | **Sessions:** 3-4

### Learning Objectives
- Understand TCP/IP networking at the systems level
- Build a protocol on top of TCP
- Handle multiple clients concurrently
- Implement serialization/deserialization
- Apply everything learned so far in a real project

### Systems Knowledge
> **TCP/IP and sockets:** TCP is a reliable, ordered byte stream built on top of IP packets. A "socket" is the OS abstraction for a network endpoint: it is a file descriptor (remember Chapter 14!) that you read/write. `bind()` assigns an address, `listen()` starts accepting connections, `accept()` returns a new socket for each client. TCP provides no message boundaries -- you receive a stream of bytes, so you need a protocol (framing) to know where messages begin and end. Common approaches: length-prefixed messages, delimiter-separated (like HTTP's `\r\n\r\n`), or fixed-size messages. Understanding this is essential for building any networked application.

### Concepts
- TCP sockets: `TcpListener`, `TcpStream`
- Network byte order
- Protocol design: framing, serialization
- `serde` for serialization
- Connection handling patterns
- Graceful shutdown
- Error recovery and reconnection
- Testing networked code

### Exercises

**Exercise 19.1 -- Echo Server**
Build a TCP echo server that accepts connections and echoes back whatever the client sends. First single-threaded (one client at a time), then multi-threaded, then async with tokio.

**Exercise 19.2 -- Protocol Design**
Design a length-prefixed binary protocol: 4-byte big-endian length header followed by a JSON payload. Implement encode/decode functions. Handle the case where a single `read()` does not return the complete message (partial reads).

**Exercise 19.3 -- Connection Pool**
Implement a client-side connection pool that maintains N persistent TCP connections to a server. Connections are checked out, used, and returned. Handle dead connections gracefully.

### Mini-Project: Distributed Key-Value Store
Build a networked key-value store with a custom binary protocol:
- **Server:** stores key-value pairs in memory, handles GET/SET/DELETE/LIST commands, supports multiple concurrent clients (async), persists data to disk (append-only log), supports basic replication (one leader, one follower)
- **Client library:** connection pooling, automatic reconnection, typed API
- **CLI client:** interactive REPL for manual queries
- **Protocol:** length-prefixed binary frames with serde serialization

This capstone project exercises networking, async I/O, serialization, error handling, file I/O, and concurrent state management.

---

## Chapter 20: Performance, Profiling, and Optimization

**Difficulty:** Advanced | **Sessions:** 2-3

### Learning Objectives
- Measure before optimizing (benchmarking with `criterion`)
- Profile CPU usage with flamegraphs
- Profile memory with heap profilers
- Understand CPU cache effects and data-oriented design
- Apply optimization techniques safely

### Systems Knowledge
> **CPU caches and performance:** Modern CPUs are fast; main memory is slow. The cache hierarchy bridges this gap. A cache line is 64 bytes -- when you access one byte, the CPU loads 64 bytes. Sequential access patterns (iterating a `Vec`) are fast because the prefetcher loads the next cache line speculatively. Random access patterns (following pointers in a linked list) cause cache misses. Data-oriented design means arranging your data for the access patterns you actually use: arrays of structs vs structs of arrays, hot/cold splitting, etc. Understanding this is the difference between code that runs and code that runs *fast*. Always measure first: intuition about performance is often wrong.

### Concepts
- Benchmarking with `criterion`
- Flamegraphs with `cargo-flamegraph`
- `perf` and `dtrace` basics
- CPU cache effects: locality, cache lines, prefetching
- Branch prediction and its effects
- SIMD basics
- Memory allocation profiling
- `#[inline]`, `#[cold]`, `likely`/`unlikely`
- Release profile tuning (`opt-level`, `lto`, `codegen-units`)
- Data-oriented design: SoA vs AoS

### Exercises

**Exercise 20.1 -- Benchmark Suite**
Take a function from a previous chapter and write `criterion` benchmarks. Optimize it and measure the improvement. Try: different algorithms, different data layouts, pre-allocation.

**Exercise 20.2 -- Cache Effects**
Write two programs that process the same data but with different access patterns: one sequential (iterate a Vec), one random (shuffle indices first). Benchmark both with varying data sizes (1KB to 100MB). Plot the results to see where cache effects become visible.

**Exercise 20.3 -- Profile and Optimize**
Take the distributed key-value store from Chapter 19 and profile it under load. Generate a flamegraph, identify the hottest functions, and optimize them. Document each optimization and its measured impact.

### Mini-Project: High-Performance Log Analyzer
Build a tool that analyzes large log files (multiple GB). Features: parse log entries, filter by date range/level/pattern, aggregate statistics (requests per second, error rates, latency percentiles). Optimize for speed: memory-mapped files, zero-copy parsing (reference original bytes instead of copying), SIMD-accelerated string search (optional), parallel processing of file chunks. Benchmark against `grep` and `awk` for simple operations. Target: process 1GB in under 2 seconds.

---

## Appendix: Recommended Resources

- **The Rust Programming Language (The Book):** https://doc.rust-lang.org/book/ -- the official guide, excellent for reference
- **Rust by Example:** https://doc.rust-lang.org/rust-by-example/ -- learn by reading examples
- **Rustlings:** https://github.com/rust-lang/rustlings -- small exercises for practice
- **Programming Rust (O'Reilly):** deep dive into the language
- **Rust for Rustaceans (Jon Gjengset):** advanced topics
- **Computer Systems: A Programmer's Perspective (CS:APP):** the best book on systems knowledge
- **Jon Gjengset's YouTube channel:** excellent advanced Rust content
- **This Week in Rust:** https://this-week-in-rust.org/ -- stay current

---

## Progress Tracking

### Current Status
- **Current Chapter:** Not started
- **Sessions Completed:** 0
- **Last Session Date:** N/A
- **Next Steps:** Begin Chapter 1

### Session Log
*(To be updated after each session)*

### Areas Needing Extra Practice
*(To be updated based on struggles)*
