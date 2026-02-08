# Rust Learning Plan: The Ruby-to-Rust Bridge

## Why This Approach Is the Best Way to Learn Rust

You have years of Ruby experience. That's not baggage -- it's a superpower, if used correctly.

Most Rust tutorials teach from scratch, ignoring what you already know. That's wasteful. You already understand closures, iterators, pattern matching, enums, generics (duck typing is a form of generic programming), error handling philosophy, and dozens of other concepts. The problem is that Rust implements these ideas with different trade-offs and constraints that serve a fundamentally different goal: **zero-cost abstractions with compile-time safety guarantees**.

This plan uses your Ruby knowledge as a **bridge, not a crutch**. For every concept, we start with "you already know this from Ruby" and then show *why* Rust does it differently. The key insight: Ruby optimizes for **developer happiness at runtime**; Rust optimizes for **correctness at compile time**. Once you internalize this single idea, everything else follows.

Here's what makes this approach powerful:

1. **Faster intuition building.** When you see Rust's `iter().map().collect()`, you already know what it *does* from Ruby's `map`. Now you just need to learn *how* Rust makes it zero-cost.

2. **Targeted mental model shifts.** Instead of learning 100 new things, you learn ~15 key shifts in thinking. Everything else maps from Ruby almost directly.

3. **Ruby anti-patterns become Rust insights.** Every time Ruby lets you shoot yourself in the foot (nil errors, thread-safety bugs, mutation surprises), Rust has a compile-time solution. Your Ruby battle scars become "aha!" moments.

4. **You'll write better Ruby too.** Understanding ownership, borrowing, and lifetimes changes how you think about all code -- you'll spot bugs in Ruby that you never would have seen before.

The danger of Ruby experience is **three specific intuitions that will mislead you**: (1) everything is an object on the heap, (2) the garbage collector handles memory so you don't think about it, and (3) mutation is free and easy. We'll address each of these head-on, early, with exercises designed to break and rebuild these intuitions.

---

## Plan Structure

Each chapter follows this pattern:
- **Ruby Anchor**: What you already know from Ruby
- **Rust Bridge**: How the concept maps (and where it diverges)
- **Mental Model Shift**: The key thinking change required
- **Aha Moments**: The insights that make it click
- **Exercises**: Small programs that port Ruby patterns to Rust
- **Mini-Project**: A hands-on project tying the chapter together

Estimated total: **15-18 chapters, ~30-50 sessions** depending on pace.

---

## Chapter 1: From `irb` to `cargo` -- Your New REPL and Toolkit

### Ruby Anchor
You know `irb`, `bundle`, `gem`, `rake`, `rspec`. Ruby's tooling is mature and convention-driven (especially Rails). You're used to `Gemfile`, `Gemfile.lock`, and `bundle exec`.

### Rust Bridge
| Ruby | Rust | Notes |
|------|------|-------|
| `irb` / `pry` | No true REPL (but `cargo run` is fast) | Rust is compiled; no eval loop |
| `bundle` / `gem` | `cargo` | Cargo is bundler + rake + gem all in one |
| `Gemfile` | `Cargo.toml` | Similar dependency declaration |
| `Gemfile.lock` | `Cargo.lock` | Same purpose: reproducible builds |
| `rake` | `cargo build`, `cargo test`, `cargo run` | Built into the tool |
| `rspec` | Built-in `#[test]` + `cargo test` | No separate test framework needed |
| `rubocop` | `clippy` + `rustfmt` | Linting + formatting |

### Mental Model Shift
Ruby is interpreted: you write code, run it, see results. Rust is compiled: there's a **compilation step** between writing and running. This step is where Rust's magic happens -- the compiler catches bugs that Ruby would only find at runtime (or in production at 3am).

### Aha Moments
- `cargo` replaces 4-5 separate Ruby tools
- Compilation errors are your friend, not your enemy -- they're free bug reports
- `cargo test` is built-in; no test framework decision paralysis
- `Cargo.toml` is cleaner than `Gemfile` because there's no `require` equivalent needed

### Exercises
1. Create a new project with `cargo new hello_rust`, explore the file structure
2. Write a function, a test, run `cargo test`
3. Add a dependency (`serde`), compare `Cargo.toml` vs `Gemfile` workflow
4. Use `clippy` and `rustfmt` -- compare to `rubocop`
5. Explore compiler error messages: intentionally write broken code and read the suggestions

### Mini-Project
**CLI Greeter**: Build a small CLI program that takes a name as argument and prints a greeting. Use `std::env::args()`. Compare to a Ruby version using `ARGV`. Introduce `cargo run -- arguments`.

---

## Chapter 2: Types -- From Duck Typing to Static Typing (and Why It's Liberating)

### Ruby Anchor
Ruby uses duck typing: if it responds to `.quack`, it's a duck. You never declare types. You rely on runtime errors and tests to catch type mismatches. You're used to `nil` being a valid value for literally anything.

### Rust Bridge
```ruby
# Ruby: no types, anything goes
def add(a, b)
  a + b  # works with integers, floats, strings... and crashes with incompatible types
end
```

```rust
// Rust: explicit types, compiler enforces correctness
fn add(a: i32, b: i32) -> i32 {
    a + b  // only works with i32, and that's the POINT
}
```

| Ruby | Rust | Key Difference |
|------|------|----------------|
| Integer (arbitrary size) | `i8, i16, i32, i64, i128, isize` | You choose the size |
| Float | `f32, f64` | You choose the precision |
| String (mutable, heap) | `String` (owned) vs `&str` (borrowed) | Two string types! |
| Symbol | No equivalent (but `&'static str` is close) | Symbols are interned strings |
| `nil` | No nil! `Option<T>` instead | **Biggest shift** |
| `true/false` | `bool` | Same idea |
| Array | `Vec<T>` (one type only) | No mixed-type arrays |
| Hash | `HashMap<K, V>` | Keys and values have fixed types |

### Mental Model Shift
In Ruby, you think "what methods does this object respond to?" In Rust, you think "what type is this, and what operations does that type support?" This feels restrictive at first but becomes liberating -- the compiler becomes your pair programmer that never misses a bug.

**The nil revolution**: In Ruby, `nil` can appear anywhere and causes `NoMethodError` at runtime. Rust's `Option<T>` makes the *possibility* of absence explicit in the type. If a function returns `Option<String>`, you MUST handle the `None` case. No more `undefined method for nil:NilClass`.

### Aha Moments
- Type inference means you don't actually write types that often: `let x = 42;` -- Rust figures out it's `i32`
- `Option<T>` eliminates the entire category of nil/null bugs
- Choosing integer sizes matters when you care about memory (which you will in systems programming)
- Two string types (`String` vs `&str`) is confusing at first but makes perfect sense once you understand ownership

### Exercises
1. **The nil killer**: Write a Ruby function that can return nil, then port it to Rust using `Option<T>`. Handle both `Some` and `None`.
2. **Type detective**: Take a Ruby hash with mixed value types and figure out how to represent it in Rust (hint: enums).
3. **String duality**: Write functions that take `&str` vs `String` parameters. Understand when to use which.
4. **Number overflow**: Demonstrate what happens when you overflow an `i8` in Rust vs Ruby's arbitrary-precision integers.
5. **Collection constraints**: Try to create a `Vec` with mixed types (like a Ruby array) and understand why it fails.

### Mini-Project
**Config Parser**: Parse a simple config file (key=value pairs) into a `HashMap<String, String>`. In Ruby this is trivial (`File.readlines.map { |l| l.split("=") }.to_h`). In Rust, handle the `Option`/`Result` types that arise from parsing. Compare error handling approaches.

---

## Chapter 3: Ownership -- The Concept Ruby Hides From You

### Ruby Anchor
In Ruby, you never think about who "owns" data. You create objects, pass them around, and the garbage collector cleans up. Multiple variables can reference the same object (`a = [1,2,3]; b = a` -- both point to the same array). You mutate freely.

```ruby
a = [1, 2, 3]
b = a           # b and a point to the SAME array
b.push(4)
puts a.inspect  # [1, 2, 3, 4] -- surprise! (or not, if you know Ruby well)
```

### Rust Bridge
```rust
let a = vec![1, 2, 3];
let b = a;          // a is MOVED to b. a is now invalid!
// println!("{:?}", a);  // COMPILE ERROR: a was moved
println!("{:?}", b);    // works fine
```

The three rules of ownership:
1. Each value has exactly **one owner**
2. When the owner goes out of scope, the value is **dropped** (freed)
3. Ownership can be **transferred** (moved) or **borrowed** (referenced)

### Mental Model Shift
**This is the single biggest shift from Ruby to Rust.** In Ruby, memory is a shared resource managed by the GC. In Rust, every piece of data has exactly one owner, and the compiler tracks ownership at compile time. This is how Rust achieves memory safety without a garbage collector.

Think of it like this: In Ruby, data is like a library book that anyone can read and write in (the library tracks who has it via GC). In Rust, data is like a physical notebook -- only one person can hold it at a time, though they can lend it to others temporarily.

### Aha Moments
- Move semantics explain why there's no GC: the compiler knows exactly when to free memory
- `Clone` is explicit copying -- Ruby's `dup`/`clone` but you MUST call it
- Small types like `i32` implement `Copy` -- they're copied automatically (like Ruby's immediate values)
- This system is what makes Rust as fast as C while being memory-safe

### Exercises
1. **The move puzzle**: Create a `String`, try to use it after moving it. Read the compiler error. Fix it with `.clone()`.
2. **Scope drops**: Create values in nested scopes, observe when they're dropped (use a struct with a `Drop` implementation).
3. **Ruby vs Rust mutation**: Port the Ruby shared-reference example above. Understand why Rust prevents the "surprise mutation" bug.
4. **Function ownership**: Write functions that take ownership vs borrow. See when values survive function calls.
5. **The clone cost**: Benchmark cloning a large vector vs borrowing. Understand why Rust makes copying explicit.

### Mini-Project
**Linked List (attempt 1)**: Try to implement a simple linked list. You WILL struggle with ownership. That's the point. This exercise reveals why self-referential data structures are hard in Rust and introduces `Box<T>` for heap allocation.

---

## Chapter 4: Borrowing and References -- Lending Without Losing

### Ruby Anchor
In Ruby, passing an object to a method gives that method access to the *same* object. There's no distinction between "I'm giving you this" and "I'm letting you look at this." Everything is a reference, always.

```ruby
def append_greeting(arr)
  arr.push("hello")  # modifies the ORIGINAL array
end
names = ["world"]
append_greeting(names)
puts names.inspect  # ["world", "hello"] -- original was mutated
```

### Rust Bridge
```rust
// Immutable borrow: you can look, but not touch
fn print_list(list: &Vec<String>) {
    println!("{:?}", list);
    // list.push("hello".to_string());  // COMPILE ERROR: can't mutate through &
}

// Mutable borrow: you can modify, but only one at a time
fn append_greeting(list: &mut Vec<String>) {
    list.push("hello".to_string());  // OK: we have &mut access
}
```

The borrowing rules:
1. You can have **many** immutable references (`&T`) OR **one** mutable reference (`&mut T`), never both
2. References must always be **valid** (no dangling pointers)

### Mental Model Shift
Think of `&` as "I'm lending you this book to read" and `&mut` as "I'm lending you this book to write in." You wouldn't lend the same book to someone for writing while others are reading it -- that would cause chaos. Rust enforces this at compile time.

**Where Ruby intuition misleads**: In Ruby, you're used to passing objects and mutating them freely. In Rust, you must explicitly opt into mutation with `&mut`, and the compiler ensures no one else is reading while you're writing. This prevents data races *by construction*.

### Aha Moments
- The borrow checker prevents the entire class of bugs where two parts of code unexpectedly share mutable state
- This is directly related to thread safety (Chapter 15) -- the same rules that prevent bugs in single-threaded code prevent data races in multi-threaded code
- `&str` is a borrowed reference to string data -- that's why there are two string types!
- Most functions should take references (`&T` or `&mut T`), not owned values

### Exercises
1. **Borrow vs move**: Rewrite Chapter 3's functions to borrow instead of taking ownership. See how the caller can keep using the value.
2. **The simultaneous borrow puzzle**: Try to hold `&` and `&mut` to the same value. Read the error. Understand why.
3. **Slice borrowing**: Use `&[T]` and `&str` slices. Compare to Ruby's ranges and string slicing.
4. **Lifetime of a reference**: Create a reference that outlives its data (dangling reference). See how the compiler catches it.
5. **Port a Ruby class**: Take a Ruby class with methods that mutate `self` and methods that read `self`. Port to Rust with `&self` and `&mut self`.

### Mini-Project
**Text Statistics**: Read a text file, compute word frequencies, longest words, sentence count, etc. Practice passing `&str` slices around without unnecessary `String` allocations. Compare memory usage intuition to a Ruby equivalent.

---

## Chapter 5: Structs and Enums -- Ruby Classes, Deconstructed

### Ruby Anchor
In Ruby, a class bundles data (instance variables) and behavior (methods). Classes are open, inheritable, and extremely flexible.

```ruby
class User
  attr_accessor :name, :email, :role

  def initialize(name, email, role = :member)
    @name = name
    @email = email
    @role = role
  end

  def admin?
    @role == :admin
  end
end
```

### Rust Bridge
Rust splits Ruby's class into two concepts:
- **Structs**: hold data (like a class with only `attr_reader`)
- **`impl` blocks**: add behavior (methods)
- **Enums**: encode variants (like Ruby's symbols on steroids)

```rust
enum Role {
    Admin,
    Member,
    Guest { expires_at: u64 },  // variants can hold data!
}

struct User {
    name: String,
    email: String,
    role: Role,
}

impl User {
    // "Constructor" (not special, just convention)
    fn new(name: String, email: String) -> Self {
        User { name, email, role: Role::Member }
    }

    fn is_admin(&self) -> bool {
        matches!(self.role, Role::Admin)
    }
}
```

### Mental Model Shift
In Ruby, a class is one thing. In Rust, data (struct/enum) and behavior (impl) are separate. There's no inheritance -- composition and traits replace it. Enums aren't just labels (like Ruby symbols); they can carry data, making them one of Rust's most powerful features.

**Where Ruby intuition helps**: Ruby's `Struct` class and `attr_reader` pattern maps directly to Rust structs. Ruby's `case/when` maps to Rust's `match`.

**Where it misleads**: No inheritance. No `method_missing`. No open classes. No monkey-patching. These constraints push you toward better designs.

### Aha Moments
- Rust enums with data replace entire design patterns (State pattern, Visitor pattern, etc.)
- `match` on enums is exhaustive -- the compiler ensures you handle every variant
- `Option<T>` is just `enum Option<T> { Some(T), None }` -- not magic, just an enum!
- `Result<T, E>` is just `enum Result<T, E> { Ok(T), Err(E) }` -- error handling is an enum!

### Exercises
1. **Port a Ruby class**: Convert a Ruby `User` class with roles to a Rust struct + enum.
2. **Enum power**: Model a payment system with `enum Payment { Card { last_four: String }, Bank { routing: String, account: String }, Crypto { wallet: String } }`. Use `match` to process each.
3. **Method syntax**: Implement `&self`, `&mut self`, and associated (static) methods. Compare to Ruby instance vs class methods.
4. **Tuple structs**: Create `struct Meters(f64)` and `struct Kilometers(f64)` -- type-safe units that Ruby can't enforce.
5. **Builder pattern**: Ruby often uses method chaining (`User.new.with_name("foo").with_role(:admin)`). Implement this in Rust.

### Mini-Project
**Task Manager CLI**: Build a to-do list CLI app with `Task` struct, `Priority` enum, `Status` enum. Support add, complete, list, and filter operations. Store in a `Vec<Task>`. Compare to how you'd build it in Ruby with plain classes.

---

## Chapter 6: Pattern Matching -- Ruby's `case` on Steroids

### Ruby Anchor
Ruby 3.x introduced pattern matching, but you're probably more familiar with `case/when`:

```ruby
case command
when "quit" then exit
when /^say (.+)/ then puts $1
when Integer then puts "got number: #{command}"
end
```

### Rust Bridge
```rust
match command {
    Command::Quit => std::process::exit(0),
    Command::Say(message) => println!("{}", message),
    Command::Repeat { times, message } => {
        for _ in 0..times {
            println!("{}", message);
        }
    }
}
```

### Mental Model Shift
In Ruby, pattern matching is optional syntactic sugar. In Rust, `match` is fundamental to the language. You use it constantly with `Option`, `Result`, enums, and destructuring. The compiler ensures every case is handled (**exhaustiveness checking**) -- no missed cases, ever.

### Aha Moments
- `if let` is syntactic sugar for matching one variant: `if let Some(x) = maybe_value { use(x) }`
- Destructuring works everywhere: function params, `let` bindings, `for` loops
- Match guards add conditions: `Some(x) if x > 0 => ...`
- The `_` wildcard is like Ruby's `else` but more powerful -- you can ignore specific parts of a structure

### Exercises
1. **Exhaustive matching**: Match on an enum, forget one variant. See the compiler error. Appreciate it.
2. **Option gymnastics**: Chain multiple `Option` operations using `match`, then simplify with combinators (`map`, `and_then`, `unwrap_or`).
3. **Destructuring fest**: Destructure nested structs, tuples, and enums in a single `match`.
4. **if let vs match**: Rewrite `match` expressions as `if let` and vice versa. Know when each is appropriate.
5. **Port Ruby case statements**: Take 3 real `case/when` blocks from Ruby code and port them to Rust `match`.

### Mini-Project
**Command Parser**: Build a simple command-line interpreter. Parse input strings into a `Command` enum with variants like `Help`, `Greet { name: String }`, `Add { a: f64, b: f64 }`, `Quit`. Use `match` to dispatch. This is the foundation for more complex parsers.

---

## Chapter 7: Error Handling -- From Exceptions to Results

### Ruby Anchor
Ruby uses exceptions for error handling:

```ruby
begin
  file = File.open("config.txt")
  data = JSON.parse(file.read)
  data.fetch("key")
rescue Errno::ENOENT => e
  puts "File not found: #{e.message}"
rescue JSON::ParserError => e
  puts "Invalid JSON: #{e.message}"
rescue KeyError
  puts "Missing key"
end
```

Exceptions can be thrown from anywhere and caught anywhere. Uncaught exceptions crash the program.

### Rust Bridge
```rust
use std::fs;

fn read_config(path: &str) -> Result<String, ConfigError> {
    let contents = fs::read_to_string(path)
        .map_err(|e| ConfigError::FileError(e.to_string()))?;
    let data: serde_json::Value = serde_json::from_str(&contents)
        .map_err(|e| ConfigError::ParseError(e.to_string()))?;
    data.get("key")
        .and_then(|v| v.as_str())
        .map(|s| s.to_string())
        .ok_or(ConfigError::MissingKey)
}
```

### Mental Model Shift
Ruby's exceptions are **invisible in the type signature** -- you can't tell from a method signature what errors it might raise. Rust's `Result<T, E>` makes errors **explicit in the return type**. Every function that can fail tells you exactly how it can fail. The `?` operator is syntactic sugar for early return on error -- similar to Ruby's early `return` but for errors specifically.

**Where Ruby intuition misleads**: You might reach for `.unwrap()` like Ruby's "just let it crash" approach. Resist this. `unwrap()` is for prototyping only. Idiomatic Rust propagates errors with `?`.

### Aha Moments
- `?` operator is the most important operator in Rust -- it replaces 90% of error handling boilerplate
- Error types compose: you can convert between error types with `From` trait
- `Result` is just an enum -- all the pattern matching skills from Chapter 6 apply
- No hidden control flow: you can always see where errors are handled by following the `?` chain
- `anyhow` crate for applications, `thiserror` crate for libraries -- know which to use when

### Exercises
1. **The `?` chain**: Write a function that reads a file, parses it, and validates content. Use `?` for each fallible step.
2. **Custom error types**: Create an error enum with `thiserror`. Compare to defining custom exception classes in Ruby.
3. **From exceptions to Results**: Take a Ruby method with `begin/rescue` and port it to Rust with `Result`.
4. **Combinators**: Chain `map`, `and_then`, `unwrap_or_else`, `ok_or` on `Result`. Compare to Ruby's `&.` (safe navigation) operator.
5. **Error context**: Use `anyhow` with `.context()` to add human-readable error messages. Compare to Ruby's exception messages.

### Mini-Project
**File Processor**: Build a program that reads multiple files, parses structured data from each (CSV or JSON), validates the data, and produces a summary report. Every step can fail differently. Practice the full error handling story: custom error types, propagation with `?`, and user-friendly error messages.

---

## Chapter 8: Traits -- From Duck Typing to Explicit Interfaces

### Ruby Anchor
Ruby uses duck typing and modules for shared behavior:

```ruby
module Printable
  def to_display
    raise NotImplementedError
  end

  def print_formatted
    puts "=== #{to_display} ==="
  end
end

class User
  include Printable
  def to_display
    "#{@name} <#{@email}>"
  end
end
```

### Rust Bridge
```rust
trait Printable {
    fn to_display(&self) -> String;  // required

    fn print_formatted(&self) {      // default implementation
        println!("=== {} ===", self.to_display());
    }
}

impl Printable for User {
    fn to_display(&self) -> String {
        format!("{} <{}>", self.name, self.email)
    }
    // print_formatted uses the default
}
```

### Mental Model Shift
Ruby modules are mixed in at runtime. Rust traits are checked at compile time. Where Ruby asks "does this object respond to this method?" Rust asks "does this type implement this trait?" The answers are determined at compile time, not runtime.

Traits replace both Ruby's duck typing AND inheritance. They're closer to Ruby modules than to classes, but with compile-time guarantees.

### Aha Moments
- Standard traits (`Display`, `Debug`, `Clone`, `PartialEq`) are like Ruby's `to_s`, `inspect`, `dup`, `==`
- `#[derive(...)]` auto-implements common traits -- like Ruby's `Comparable` mixin but automatic
- Trait bounds (`fn process<T: Printable>(item: T)`) are explicit duck typing
- You can implement your traits for external types AND external traits for your types (orphan rule limits this)
- Traits with generics + associated types replace a lot of Ruby metaprogramming

### Exercises
1. **Port a Ruby module**: Convert a Ruby module with `include`/`extend` to a Rust trait.
2. **Standard traits**: Implement `Display`, `Debug`, `PartialEq`, `Clone` for a custom type. Compare to Ruby's `to_s`, `inspect`, `==`, `dup`.
3. **Trait bounds**: Write generic functions that accept "anything Printable." Compare to Ruby's implicit duck typing.
4. **Default methods**: Create traits with default implementations. Override selectively. Compare to Ruby module methods.
5. **The orphan rule**: Try to implement an external trait for an external type. Understand the newtype pattern workaround.

### Mini-Project
**Shape Calculator**: Define a `Shape` trait with `area()` and `perimeter()`. Implement for `Circle`, `Rectangle`, `Triangle`. Add a `Drawable` trait. Combine them. Use trait objects (`dyn Shape`) for a heterogeneous collection. Compare to Ruby's polymorphism.

---

## Chapter 9: Generics -- Making Duck Typing Compile-Safe

### Ruby Anchor
Ruby's generics are implicit through duck typing:

```ruby
def first_or_default(collection, default)
  collection.first || default  # works with any collection, any default
end
```

### Rust Bridge
```rust
fn first_or_default<T: Clone>(collection: &[T], default: &T) -> T {
    collection.first().cloned().unwrap_or_else(|| default.clone())
}
```

### Mental Model Shift
Ruby: "I'll accept anything and hope it works." Rust: "I'll accept anything that provably has the capabilities I need." Generics + trait bounds are compile-time duck typing. You get the flexibility of duck typing with the safety of static typing.

### Aha Moments
- Generics are zero-cost: `Vec<i32>` and `Vec<String>` are compiled to specialized code (monomorphization)
- Trait bounds are explicit requirements: `T: Display + Clone` means "T must be displayable and cloneable"
- `impl Trait` in function parameters is shorthand for simple generic bounds
- Rust's turbofish `::<Type>` is like Ruby's... nothing. Ruby doesn't need it because types are resolved at runtime.

### Exercises
1. **Generic functions**: Write `max_by_key` that works for any type with `Ord`. Compare to Ruby's `max_by`.
2. **Generic structs**: Create `Stack<T>` and `Queue<T>`. Ruby doesn't need the `<T>` but Rust does -- understand why.
3. **Bound stacking**: Write a function that requires `T: Display + Clone + PartialOrd`. See how bounds compose.
4. **Where clauses**: Rewrite complex bounds using `where` for readability.
5. **Monomorphization**: Use generics vs trait objects. Understand the performance difference.

### Mini-Project
**Generic Cache**: Build a `Cache<K, V>` struct with `get`, `set`, `evict` methods. Support TTL (time-to-live). This is like Ruby's `Hash` but type-safe and with explicit constraints on keys (`Hash + Eq`) and values.

---

## Chapter 10: Iterators and Closures -- Where Ruby and Rust Converge

### Ruby Anchor
This is where your Ruby knowledge shines brightest. Ruby's iterator pattern is legendary:

```ruby
users
  .select { |u| u.active? }
  .map { |u| u.name.upcase }
  .sort
  .take(10)
```

### Rust Bridge
```rust
users.iter()
    .filter(|u| u.is_active())
    .map(|u| u.name.to_uppercase())
    .sorted()  // from itertools crate
    .take(10)
    .collect::<Vec<_>>()
```

### Mental Model Shift
The API is remarkably similar! Key differences:
1. Rust iterators are **lazy** by default (Ruby's `Enumerator::Lazy` equivalent)
2. You must `.collect()` to materialize results (no implicit conversion)
3. Closures have three types based on how they capture variables (`Fn`, `FnMut`, `FnOnce`) -- corresponding to `&self`, `&mut self`, and ownership
4. Iterator adapters are **zero-cost** -- the compiler optimizes chains into a single loop

### Aha Moments
- Rust's `iter()` / `into_iter()` / `iter_mut()` distinguish borrowing from ownership
- `collect()` is incredibly powerful -- it can collect into `Vec`, `HashMap`, `String`, `Result<Vec<T>, E>`, and more
- Closures capturing variables follow ownership rules -- `move ||` forces ownership transfer
- Iterator chains compile to the same code as hand-written loops (zero-cost abstraction!)

### Exercises
1. **Direct translation**: Port 5 Ruby iterator chains to Rust. Note the differences.
2. **Lazy vs eager**: Compare `(0..1_000_000).filter(...).take(5)` behavior.
3. **Closure captures**: Write closures that capture by reference, mutable reference, and ownership. Understand `Fn` vs `FnMut` vs `FnOnce`.
4. **Custom iterator**: Implement `Iterator` trait for a custom type. Compare to Ruby's `Enumerable`.
5. **collect() magic**: Collect into different types (`Vec`, `HashMap`, `String`, `Result<Vec<T>, E>`).

### Mini-Project
**Data Pipeline**: Build a log file analyzer. Read lines, parse structured data, filter, transform, aggregate statistics. Chain iterators for the entire pipeline. Compare performance and readability to a Ruby version.

---

## Chapter 11: Lifetimes -- The Concept Ruby Doesn't Have

### Ruby Anchor
Ruby has no equivalent. The GC handles object lifetimes automatically. The closest mental model is Ruby's block-scoping, but it's a weak analogy.

### Rust Bridge
```rust
// This won't compile:
fn longest(a: &str, b: &str) -> &str {  // ERROR: lifetime of return value?
    if a.len() > b.len() { a } else { b }
}

// This will:
fn longest<'a>(a: &'a str, b: &'a str) -> &'a str {
    if a.len() > b.len() { a } else { b }
}
```

### Mental Model Shift
Lifetimes answer the question: "How long does this reference remain valid?" In Ruby, objects live as long as someone references them (GC). In Rust, the compiler needs to prove at compile time that no reference outlives its data. Lifetime annotations don't change how long things live -- they just help the compiler verify correctness.

### Aha Moments
- Most lifetimes are inferred (lifetime elision rules handle ~90% of cases)
- `'static` means "lives for the entire program" (like string literals)
- Lifetime errors mean you're trying to use a reference to freed data -- the compiler is saving you from a segfault
- Structs holding references need lifetime annotations: `struct Config<'a> { name: &'a str }`

### Exercises
1. **Trigger a lifetime error**: Write code that creates a dangling reference. Read the error carefully.
2. **Annotate lifetimes**: Add lifetime annotations to functions that return references. Understand elision rules.
3. **Struct lifetimes**: Create a struct that borrows data. Understand why and when this is useful vs owning data.
4. **Multiple lifetimes**: Functions with different lifetime parameters. Understand `'a` vs `'b`.
5. **Static lifetime**: Understand `&'static str` and when to use it.

### Mini-Project
**Zero-Copy Parser**: Build a parser that borrows from the input string instead of allocating new strings. Parse a CSV-like format where fields are `&str` slices into the original input. Compare memory usage to a Ruby version that creates new String objects for each field.

---

## Chapter 12: Smart Pointers and Interior Mutability

### Ruby Anchor
Every Ruby object reference is essentially a smart pointer managed by the GC. Ruby's `ObjectSpace` and reference counting (for some implementations) are hidden from you.

### Rust Bridge
| Concept | Ruby | Rust |
|---------|------|------|
| Heap allocation | Automatic (most objects) | `Box<T>` (explicit) |
| Shared ownership | GC handles it | `Rc<T>` (reference counting) |
| Thread-safe sharing | GIL handles it | `Arc<T>` (atomic ref counting) |
| Interior mutability | Default (everything is mutable) | `Cell<T>`, `RefCell<T>`, `Mutex<T>` |

### Mental Model Shift
In Ruby, the GC provides infinite flexibility: anything can reference anything, and it all gets cleaned up. Rust makes you choose your ownership strategy explicitly. This is more work but gives you precise control over memory layout and performance.

`RefCell<T>` is the closest to Ruby's "everything is mutable" model -- it moves borrow checking from compile time to runtime. Use it sparingly; prefer compile-time checking when possible.

### Aha Moments
- `Box<T>` is just a heap allocation -- like Ruby's implicit heap allocation but explicit
- `Rc<T>` implements Ruby-style shared ownership without a full GC
- `Rc<RefCell<T>>` is "Ruby mode" -- shared mutable state with runtime checking
- `Arc<Mutex<T>>` is the thread-safe version -- what Ruby's GIL gives you implicitly, Rust makes explicit

### Exercises
1. **Box for recursion**: Implement a recursive data structure (tree, linked list) using `Box<T>`.
2. **Rc for sharing**: Create a graph structure with `Rc<T>`. Compare to Ruby's automatic sharing.
3. **RefCell for mutability**: Use `Rc<RefCell<T>>` to create Ruby-like shared mutable state. Feel the unsafety.
4. **Reference cycles**: Create a cycle with `Rc`. See the memory leak. Fix with `Weak<T>`.
5. **Choose the right pointer**: Given 5 scenarios, pick the correct smart pointer type.

### Mini-Project
**DOM Tree**: Build a simple HTML-like tree structure. Nodes have children and a parent reference. This requires `Rc`, `Weak`, and `RefCell`. Compare to how trivially easy this is in Ruby (and understand the trade-offs).

---

## Chapter 13: Closures, Function Pointers, and Callbacks

### Ruby Anchor
Ruby has blocks, procs, lambdas, and method objects. Closures are everywhere:

```ruby
multiplier = ->(factor) { ->(x) { x * factor } }
double = multiplier.(2)
puts double.(5)  # 10
```

### Rust Bridge
```rust
fn multiplier(factor: i32) -> impl Fn(i32) -> i32 {
    move |x| x * factor
}

let double = multiplier(2);
println!("{}", double(5));  // 10
```

### Mental Model Shift
Ruby closures always capture by reference and the GC keeps everything alive. Rust closures must declare *how* they capture: by reference (`Fn`), by mutable reference (`FnMut`), or by ownership (`FnOnce`). The `move` keyword forces ownership capture.

### Aha Moments
- `Fn` traits map to Ruby's proc/lambda distinction but based on capture semantics
- Returning closures requires `impl Fn(...)` or `Box<dyn Fn(...)>` -- because closures have unique anonymous types
- `move` closures are essential for threading -- they take ownership of captured variables
- Higher-order functions in Rust are zero-cost thanks to monomorphization

### Exercises
1. **Closure types**: Write closures that are `Fn`, `FnMut`, and `FnOnce`. Understand which is which.
2. **Closure factories**: Build functions that return closures (like Ruby's `multiplier` above).
3. **Callbacks**: Implement a callback system. Compare to Ruby blocks and `yield`.
4. **Move semantics**: Use `move` closures with threads. Understand why `move` is needed.
5. **Storing closures**: Store closures in structs and vectors using `Box<dyn Fn(...)>`.

### Mini-Project
**Event System**: Build a simple event emitter (like Ruby's `Observable` or Node's `EventEmitter`). Register callbacks, emit events, handle event data. Practice storing and invoking closures.

---

## Chapter 14: Modules, Crates, and Visibility -- From Gems to Crates

### Ruby Anchor
Ruby has `require`, modules for namespacing, `gem` for packages, and a fairly loose visibility model (`public`, `private`, `protected`).

### Rust Bridge
| Ruby | Rust |
|------|------|
| `require "library"` | `use crate_name::module::Type;` |
| Module (namespace) | `mod module_name { }` |
| `gem` | crate (library) |
| `Gemfile` | `Cargo.toml [dependencies]` |
| `private` | default (everything private unless `pub`) |
| `protected` | `pub(crate)`, `pub(super)` |
| `public` | `pub` |

### Mental Model Shift
Ruby defaults to public; Rust defaults to private. Ruby's `require` is runtime; Rust's `use` is compile-time with a full module tree. Rust's module system is more like a file system hierarchy than Ruby's flat namespace.

### Aha Moments
- Everything is private by default -- the opposite of Ruby
- `pub(crate)` is like Ruby's `protected` but actually useful
- One file per module (or `mod.rs` for directories) keeps things organized
- The `use` statement is just path shortening, not code loading

### Exercises
1. **Module hierarchy**: Split a program into modules. Practice `pub`, `pub(crate)`, and private.
2. **Re-exports**: Use `pub use` to create a clean public API. Compare to Ruby's explicit module includes.
3. **External crates**: Add dependencies, use them. Compare workflow to `bundle add`.
4. **Workspaces**: Create a multi-crate project. Compare to a multi-gem Ruby project.
5. **Feature flags**: Use cargo features for optional functionality. Compare to Ruby's optional gem groups.

### Mini-Project
**Library + Binary**: Create a project with a library crate and a binary crate that uses it. The library provides a clean public API. Practice good module organization.

---

## Chapter 15: Concurrency -- What the GIL Was Hiding

### Ruby Anchor
Ruby has threads, but the GIL (Global Interpreter Lock) means only one thread executes Ruby code at a time. You've probably used threading for IO-bound work or relied on multi-process concurrency (Puma, Sidekiq).

```ruby
# Ruby: threads exist but GIL limits parallelism
threads = 10.times.map do |i|
  Thread.new { expensive_computation(i) }
end
threads.each(&:join)
# Not actually parallel for CPU-bound work!
```

### Rust Bridge
```rust
use std::thread;

// Rust: true parallelism, compile-time safety
let handles: Vec<_> = (0..10)
    .map(|i| thread::spawn(move || expensive_computation(i)))
    .collect();

let results: Vec<_> = handles.into_iter()
    .map(|h| h.join().unwrap())
    .collect();
```

### Mental Model Shift
**This is where Rust's ownership system pays off the most.** The same rules that seemed restrictive in Chapters 3-4 now prevent data races at compile time. Rust achieves what Ruby's GIL achieves (thread safety) but without sacrificing parallelism. If your Rust code compiles, it's free of data races. Period.

### Aha Moments
- The borrow checker IS the concurrency checker -- same rules, different context
- `Send` and `Sync` traits are compiler-checked markers for thread safety
- `Arc<Mutex<T>>` is the Rust equivalent of Ruby's GIL-protected global state, but fine-grained
- `rayon` crate gives you parallel iterators: `data.par_iter().map(...)` -- parallel Ruby-style iteration!
- Channels (`mpsc`) are like Ruby's `Queue` but type-safe

### Exercises
1. **True parallelism**: CPU-bound work with threads. Benchmark vs Ruby. See the real speedup.
2. **Shared state**: Use `Arc<Mutex<T>>` for shared counters. Compare to Ruby's `Mutex`.
3. **Channels**: Use `mpsc` channels for producer/consumer. Compare to Ruby's `Queue`.
4. **Rayon parallel iterators**: `par_iter()` for data parallelism. It's Ruby-like iteration but parallel.
5. **Deadlock prevention**: Create a potential deadlock scenario. Understand why Rust can't prevent all deadlocks (but prevents data races).

### Mini-Project
**Parallel Web Scraper**: Fetch multiple URLs in parallel, parse content, aggregate results. Compare to a Ruby threaded version. Measure the performance difference. Use channels for communication.

---

## Chapter 16: Async/Await -- Beyond Ruby's Fibers

### Ruby Anchor
Ruby has Fibers for cooperative concurrency and `async` gems. Rails 7+ has some async support. But async Rust is a different beast.

### Rust Bridge
```rust
// Using tokio runtime
#[tokio::main]
async fn main() {
    let results = futures::future::join_all(
        urls.iter().map(|url| fetch_url(url))
    ).await;
}

async fn fetch_url(url: &str) -> Result<String, reqwest::Error> {
    reqwest::get(url).await?.text().await
}
```

### Mental Model Shift
Ruby's async is optional and relatively new. Rust's async is fundamental to network services. The key concept: `async fn` returns a `Future` that does nothing until `.await`ed (lazy, like Rust iterators). You need a runtime (`tokio`, `async-std`) to drive futures.

### Aha Moments
- `async`/`await` is syntactic sugar for state machines the compiler generates
- Tokio is like Ruby's EventMachine/Async but built into the language
- Pin and Unpin relate to self-referential futures (advanced, but important to know about)
- Async Rust is where the language gets hardest -- don't worry if it takes time

### Exercises
1. **Basic async**: Fetch multiple URLs concurrently with `tokio` and `reqwest`.
2. **Async channels**: Build an async producer/consumer pipeline.
3. **Timeouts**: Add timeouts to async operations. Compare to Ruby's `Timeout` module.
4. **Streams**: Use async streams. Compare to Ruby's lazy enumerators.
5. **Error handling in async**: Propagate errors through async chains.

### Mini-Project
**Async Chat Server**: Build a simple TCP chat server using `tokio`. Multiple clients connect and broadcast messages. This is a classic network programming exercise that showcases async Rust's strengths.

---

## Chapter 17: Unsafe Rust and FFI -- Talking to C (and Ruby!)

### Ruby Anchor
Ruby's C extensions and FFI (Foreign Function Interface) let you call C code. You may have used gems that wrap C libraries.

### Rust Bridge
```rust
// Calling C from Rust
extern "C" {
    fn strlen(s: *const c_char) -> usize;
}

// Exposing Rust to C (and therefore to Ruby!)
#[no_mangle]
pub extern "C" fn fast_computation(input: i32) -> i32 {
    // Rust implementation callable from C, Python, Ruby...
    input * 2
}
```

### Mental Model Shift
`unsafe` doesn't mean "dangerous" -- it means "I'm taking responsibility for invariants the compiler can't check." It's like Ruby's `send(:private_method)` -- bypassing safety for a specific reason.

### Aha Moments
- `unsafe` is an audit marker, not a danger sign -- it narrows where bugs can hide
- Rust can replace C extensions in Ruby gems (with better safety!)
- The `magnus` or `rutie` crate lets you write Ruby gems in Rust
- FFI is how Rust integrates with the existing ecosystem

### Exercises
1. **Raw pointers**: Work with `*const T` and `*mut T`. Understand why they need `unsafe`.
2. **Call C from Rust**: Link a C library and call its functions.
3. **Expose Rust to C**: Create a shared library callable from C.
4. **Write a Ruby gem in Rust**: Use `magnus` to create a native Ruby extension in Rust.
5. **Safe wrappers**: Wrap unsafe FFI in safe Rust APIs. Understand the pattern.

### Mini-Project
**Ruby Gem in Rust**: Build a Ruby gem that uses Rust for the heavy computation. Something like a fast string similarity algorithm or JSON parser. Benchmark against the pure Ruby version.

---

## Chapter 18: Capstone Project -- Full Application

### The Final Challenge
Build a substantial application that combines everything learned. Choose one:

**Option A: HTTP API Server**
Build a REST API with `axum` or `actix-web`, database access (`sqlx`), authentication, and structured error handling. Compare the architecture to a Rails API. Deploy it.

**Option B: CLI Tool Suite**
Build a multi-command CLI tool (like `git` or `cargo`) using `clap`. File operations, network requests, configuration management, parallel processing. Publish to crates.io.

**Option C: Systems Tool**
Build a log aggregator, file watcher, or process manager. Real systems programming with files, processes, signals, and networking.

### What This Tests
- Ownership and borrowing in a real codebase
- Error handling across module boundaries
- Trait-based architecture
- Async programming for IO
- Testing and documentation
- Publishing and distribution

---

## Ruby vs Rust Quick Reference

| Ruby Concept | Rust Equivalent | Chapter |
|-------------|----------------|---------|
| `nil` | `Option<T>` | 2 |
| `begin/rescue` | `Result<T, E>` + `?` | 7 |
| `.each` / `.map` / `.select` | `.iter().for_each()` / `.map()` / `.filter()` | 10 |
| Blocks / Procs / Lambdas | Closures (`Fn`, `FnMut`, `FnOnce`) | 13 |
| Duck typing | Trait bounds (`T: Display`) | 8, 9 |
| `Class` | `struct` + `impl` | 5 |
| Inheritance | Traits (composition over inheritance) | 8 |
| `module` (mixin) | `trait` (with default methods) | 8 |
| `module` (namespace) | `mod` | 14 |
| GC | Ownership + borrowing | 3, 4 |
| `Mutex` | `Mutex<T>` (but no GIL!) | 15 |
| `Thread.new` | `std::thread::spawn` (true parallelism) | 15 |
| Gems | Crates | 14 |
| `attr_accessor` | `pub` fields + methods | 5 |
| `method_missing` | No equivalent (and that's good) | -- |
| Monkey patching | Extension traits (controlled) | 8 |

## Key Mental Model Shifts (Summary)

1. **From GC to Ownership**: You manage memory through ownership rules, not garbage collection
2. **From Runtime to Compile-time**: Errors caught at compile time, not in production
3. **From Duck Typing to Trait Bounds**: Same flexibility, but verified at compile time
4. **From Exceptions to Results**: Errors are values, not control flow
5. **From "Everything Mutable" to "Mutation is Explicit"**: `&mut` must be requested and is exclusive
6. **From GIL to True Concurrency**: Real parallelism with compile-time safety
7. **From Implicit to Explicit**: Copying, allocation, error handling -- all visible in code

---

## Progress Tracking

| Chapter | Status | Sessions | Notes |
|---------|--------|----------|-------|
| 1. Tooling | Not started | -- | -- |
| 2. Types | Not started | -- | -- |
| 3. Ownership | Not started | -- | -- |
| 4. Borrowing | Not started | -- | -- |
| 5. Structs/Enums | Not started | -- | -- |
| 6. Pattern Matching | Not started | -- | -- |
| 7. Error Handling | Not started | -- | -- |
| 8. Traits | Not started | -- | -- |
| 9. Generics | Not started | -- | -- |
| 10. Iterators/Closures | Not started | -- | -- |
| 11. Lifetimes | Not started | -- | -- |
| 12. Smart Pointers | Not started | -- | -- |
| 13. Closures Deep Dive | Not started | -- | -- |
| 14. Modules/Crates | Not started | -- | -- |
| 15. Concurrency | Not started | -- | -- |
| 16. Async | Not started | -- | -- |
| 17. Unsafe/FFI | Not started | -- | -- |
| 18. Capstone | Not started | -- | -- |
