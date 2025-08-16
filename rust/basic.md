# Rust Syntax Cheatsheet for C++ Developers

## Table of Contents
1. [Basic Syntax](#basic-syntax)
2. [Variables & Data Types](#variables--data-types)
3. [Functions](#functions)
4. [Control Flow](#control-flow)
5. [Ownership & Borrowing](#ownership--borrowing)
6. [Structs & Enums](#structs--enums)
7. [Pattern Matching](#pattern-matching)
8. [Error Handling](#error-handling)
9. [Collections](#collections)
10. [Iterators](#iterators)
11. [Traits](#traits)
12. [Modules](#modules)
13. [Memory Management](#memory-management)
14. [Concurrency](#concurrency)

## Basic Syntax

### Hello World
```rust
fn main() {
    println!("Hello, world!");  // println! is a macro, not function
}
```

### Comments
```rust
// Single line comment
/* Block comment */
/// Documentation comment for items
//! Inner documentation comment
```

### Semicolons
```rust
// Statements end with semicolons
let x = 5;
let y = 6;

// Expressions don't (and return values)
let z = {
    let a = 3;
    let b = 4;
    a + b  // No semicolon = return value
};
```

## Variables & Data Types

### Variable Declaration
```rust
// Immutable by default (unlike C++)
let x = 5;              // Type inferred
let x: i32 = 5;         // Explicit type
let mut y = 10;         // Mutable variable
const PI: f64 = 3.14;   // Compile-time constant

// Shadowing (redefining variables)
let x = 5;
let x = x + 1;          // Valid! Creates new variable
let x = "hello";        // Can even change type
```

### Primitive Types
```rust
// Integers (signed/unsigned)
let a: i8 = -128;       // 8-bit signed
let b: u8 = 255;        // 8-bit unsigned
let c: i32 = -2147483648; // 32-bit signed (default)
let d: u64 = 18446744073709551615; // 64-bit unsigned

// Floating point
let e: f32 = 3.14;      // 32-bit
let f: f64 = 2.718;     // 64-bit (default)

// Boolean & Character
let g: bool = true;
let h: char = 'A';      // Unicode scalar (4 bytes)

// Arrays & Tuples
let arr: [i32; 5] = [1, 2, 3, 4, 5];
let tup: (i32, f64, char) = (42, 3.14, 'x');
```

### Strings
```rust
// String slice (&str) - immutable reference
let s1: &str = "Hello";         // String literal
let s2 = "World";               // Type inferred

// String - heap allocated, mutable
let mut s3 = String::new();
let s4 = String::from("Hello");
let s5 = "Hello".to_string();

// String operations
s3.push_str("Hello");
s3.push('!');
let s6 = format!("{} {}", s4, s2);
```

## Functions

### Basic Functions
```rust
fn add(x: i32, y: i32) -> i32 {
    x + y  // No return keyword needed (expression)
}

fn print_number(x: i32) {
    println!("Number: {}", x);
}

// Early return
fn divide(x: f64, y: f64) -> f64 {
    if y == 0.0 {
        return 0.0;  // Early return needs semicolon
    }
    x / y
}
```

### Function Pointers & Closures
```rust
// Function pointer
fn square(x: i32) -> i32 { x * x }
let f: fn(i32) -> i32 = square;

// Closures (like lambdas)
let add_one = |x: i32| x + 1;
let multiply = |x, y| x * y;  // Types inferred

// Capturing environment
let factor = 2;
let scale = |x| x * factor;  // Captures factor
```

## Control Flow

### If Statements
```rust
let number = 6;

// If is an expression (returns value)
let result = if number % 2 == 0 {
    "even"
} else {
    "odd"
};

// No parentheses needed around condition
if number < 5 {
    println!("Small");
} else if number < 10 {
    println!("Medium");
} else {
    println!("Large");
}
```

### Loops
```rust
// Infinite loop
loop {
    println!("Forever!");
    break;  // Exit loop
}

// While loop
let mut n = 0;
while n < 3 {
    println!("{}", n);
    n += 1;
}

// For loop (iterator-based)
for i in 0..5 {          // Range 0 to 4
    println!("{}", i);
}

for i in 0..=5 {         // Range 0 to 5 (inclusive)
    println!("{}", i);
}

let arr = [1, 2, 3, 4, 5];
for element in arr.iter() {
    println!("{}", element);
}

// Loop labels (for nested loops)
'outer: loop {
    loop {
        break 'outer;    // Break outer loop
    }
}
```

## Ownership & Borrowing

### Ownership Rules
```rust
// 1. Each value has one owner
// 2. When owner goes out of scope, value is dropped
// 3. Only one owner at a time

let s1 = String::from("hello");
let s2 = s1;  // s1 moved to s2, s1 no longer valid
// println!("{}", s1);  // Error! s1 moved

// Clone for deep copy
let s3 = s2.clone();
println!("{} {}", s2, s3);  // Both valid
```

### References & Borrowing
```rust
fn calculate_length(s: &String) -> usize {
    s.len()  // s is a reference, doesn't own the value
}

let s = String::from("hello");
let len = calculate_length(&s);  // Borrow s
println!("{} has length {}", s, len);  // s still valid

// Mutable references
fn change(s: &mut String) {
    s.push_str(", world");
}

let mut s = String::from("hello");
change(&mut s);

// Rules: Either one mutable reference OR any number of immutable references
let r1 = &s;      // OK
let r2 = &s;      // OK
// let r3 = &mut s;  // Error! Can't have mutable ref with immutable refs
```

### Lifetimes
```rust
// Lifetime annotations
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

// Struct with references needs lifetime
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

## Structs & Enums

### Structs
```rust
// Classic struct
struct User {
    username: String,
    email: String,
    active: bool,
    sign_in_count: u64,
}

// Create instance
let user1 = User {
    email: String::from("test@example.com"),
    username: String::from("user123"),
    active: true,
    sign_in_count: 1,
};

// Tuple structs
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);

// Unit struct (no fields)
struct AlwaysEqual;
```

### Implementation Blocks
```rust
impl User {
    // Associated function (like static method)
    fn new(email: String, username: String) -> User {
        User {
            email,
            username,
            active: true,
            sign_in_count: 1,
        }
    }
    
    // Method (takes &self)
    fn is_active(&self) -> bool {
        self.active
    }
    
    // Mutable method
    fn deactivate(&mut self) {
        self.active = false;
    }
    
    // Takes ownership
    fn delete(self) {
        // User is consumed here
    }
}

let mut user = User::new(
    String::from("test@example.com"),
    String::from("user123")
);
```

### Enums
```rust
// Simple enum
enum Direction {
    North,
    South,
    East,
    West,
}

// Enum with data
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

// Option enum (built-in, like nullable)
let some_number = Some(5);
let absent_number: Option<i32> = None;

// Result enum (for error handling)
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

## Pattern Matching

### Match Expression
```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),  // Default case
}

// Match with values
let result = match x {
    1 => "one",
    2 => "two", 
    _ => "other",
};

// Match enums
let msg = Message::Write(String::from("hello"));
match msg {
    Message::Quit => println!("Quit"),
    Message::Move { x, y } => println!("Move to ({}, {})", x, y),
    Message::Write(text) => println!("Write: {}", text),
    Message::ChangeColor(r, g, b) => println!("Color: ({}, {}, {})", r, g, b),
}

// Match Option
let x: Option<i32> = Some(5);
match x {
    Some(i) => println!("Got: {}", i),
    None => println!("Nothing"),
}
```

### If Let & While Let
```rust
// If let (shorthand for single pattern match)
let some_value = Some(3);
if let Some(x) = some_value {
    println!("Got: {}", x);
}

// While let
let mut stack = Vec::new();
stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

## Error Handling

### Result Type
```rust
use std::fs::File;
use std::io::ErrorKind;

// Function that can fail
fn divide(x: f64, y: f64) -> Result<f64, String> {
    if y == 0.0 {
        Err(String::from("Division by zero"))
    } else {
        Ok(x / y)
    }
}

// Handle Result
match divide(10.0, 2.0) {
    Ok(result) => println!("Result: {}", result),
    Err(error) => println!("Error: {}", error),
}
```

### Question Mark Operator
```rust
use std::fs::File;
use std::io::{self, Read};

// ? operator propagates errors
fn read_file_content(filename: &str) -> Result<String, io::Error> {
    let mut file = File::open(filename)?;  // Returns error if fails
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;   // Returns error if fails
    Ok(contents)  // Success case
}
```

### Panic
```rust
// Unrecoverable errors
panic!("Something went wrong!");

// Unwrap (panics on error)
let f = File::open("file.txt").unwrap();

// Expect (panic with custom message)
let f = File::open("file.txt").expect("Failed to open file");
```

## Collections

### Vectors
```rust
// Create vector
let mut v: Vec<i32> = Vec::new();
let v2 = vec![1, 2, 3, 4, 5];  // Macro

// Add elements
v.push(1);
v.push(2);

// Access elements
let third = &v[2];           // Panics if out of bounds
let third = v.get(2);        // Returns Option<&T>

match v.get(2) {
    Some(third) => println!("Third element: {}", third),
    None => println!("No third element"),
}

// Iterate
for i in &v {
    println!("{}", i);
}

for i in &mut v {
    *i += 50;  // Modify elements
}
```

### HashMap
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// Get value
let team_name = String::from("Blue");
let score = scores.get(&team_name);  // Returns Option<&V>

// Iterate
for (key, value) in &scores {
    println!("{}: {}", key, value);
}

// Update values
scores.entry(String::from("Blue")).or_insert(0);  // Insert if not exists
*scores.entry(String::from("Blue")).or_insert(0) += 10;  // Update
```

## Iterators

### Iterator Basics
```rust
let v = vec![1, 2, 3, 4, 5];

// Create iterator
let iter = v.iter();  // Iterator over references

// Iterator methods (lazy)
let doubled: Vec<i32> = v.iter().map(|x| x * 2).collect();
let sum: i32 = v.iter().sum();
let filtered: Vec<&i32> = v.iter().filter(|&x| *x > 2).collect();

// For each
v.iter().for_each(|x| println!("{}", x));

// Find
let found = v.iter().find(|&x| *x > 3);  // Returns Option
```

### Iterator Adaptors
```rust
let v = vec![1, 2, 3, 4, 5];

// Chain iterators
let v2 = vec![6, 7, 8];
let combined: Vec<i32> = v.iter().chain(v2.iter()).cloned().collect();

// Enumerate (with index)
for (i, value) in v.iter().enumerate() {
    println!("{}: {}", i, value);
}

// Zip
let names = vec!["Alice", "Bob", "Charlie"];
let ages = vec![30, 25, 35];
for (name, age) in names.iter().zip(ages.iter()) {
    println!("{} is {} years old", name, age);
}
```

## Traits

### Defining Traits
```rust
// Like interfaces in other languages
trait Summary {
    fn summarize(&self) -> String;
    
    // Default implementation
    fn summarize_author(&self) -> String {
        format!("(Read more from {}...)", self.summarize())
    }
}

struct Article {
    headline: String,
    content: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}: {}", self.headline, self.content)
    }
}
```

### Common Traits
```rust
// Debug - allows printing with {:?}
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

// Clone - allows .clone()
#[derive(Clone)]
struct User {
    name: String,
}

// Copy - allows simple bit-wise copying (for stack types)
#[derive(Copy, Clone)]
struct Point2D {
    x: f64,
    y: f64,
}

// PartialEq - allows == and !=
#[derive(PartialEq)]
struct Person {
    name: String,
    age: u32,
}
```

### Trait Bounds
```rust
// Function that accepts any type implementing Display
use std::fmt::Display;

fn print_it<T: Display>(item: T) {
    println!("{}", item);
}

// Multiple bounds
fn compare<T: Display + PartialOrd>(a: T, b: T) {
    if a > b {
        println!("{} > {}", a, b);
    }
}

// Where clause (cleaner syntax)
fn some_function<T, U>(t: T, u: U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // Implementation
    0
}
```

## Modules

### Module System
```rust
// lib.rs or main.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        fn seat_at_table() {}  // Private
    }
    
    mod serving {
        fn take_order() {}
        fn serve_order() {}
    }
}

// Use keyword
use crate::front_of_house::hosting;
use std::collections::HashMap;
use std::io::{self, Write};  // Multiple imports

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

### External Files
```rust
// main.rs
mod utils;  // Looks for utils.rs or utils/mod.rs

use utils::helper_function;

// utils.rs
pub fn helper_function() {
    println!("Helper!");
}
```

## Memory Management

### Stack vs Heap
```rust
// Stack allocated (Copy types)
let x = 5;        // i32 on stack
let y = x;        // Copy, both valid

// Heap allocated  
let s1 = String::from("hello");  // String on heap
let s2 = s1;      // Move, s1 invalid

// Box - heap allocation
let b = Box::new(5);  // i32 on heap
```

### Reference Counting
```rust
use std::rc::Rc;
use std::cell::RefCell;

// Rc - Reference counted (single threaded)
let data = Rc::new(vec![1, 2, 3]);
let data2 = Rc::clone(&data);  // Increment ref count

// RefCell - Interior mutability
let x = RefCell::new(5);
*x.borrow_mut() = 10;  // Runtime borrow checking

// Combining Rc and RefCell
let shared_data = Rc::new(RefCell::new(vec![1, 2, 3]));
```

## Concurrency

### Threads
```rust
use std::thread;
use std::time::Duration;

// Spawn thread
let handle = thread::spawn(|| {
    for i in 1..10 {
        println!("Thread: {}", i);
        thread::sleep(Duration::from_millis(1));
    }
});

// Wait for thread
handle.join().unwrap();

// Move closure (transfer ownership)
let v = vec![1, 2, 3];
let handle = thread::spawn(move || {
    println!("Vector: {:?}", v);
});
```

### Channels
```rust
use std::sync::mpsc;  // Multiple producer, single consumer

let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let val = String::from("Hello");
    tx.send(val).unwrap();
});

let received = rx.recv().unwrap();
println!("Received: {}", received);
```

### Shared State
```rust
use std::sync::{Arc, Mutex};
use std::thread;

// Arc - Atomic reference counting (thread-safe)
// Mutex - Mutual exclusion
let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap());
```

## Key Differences from C++

### Memory Safety
- **No manual memory management** - no `new`/`delete`, `malloc`/`free`
- **Ownership system** prevents use-after-free, double-free, memory leaks
- **Borrowing** instead of pointers/references with manual lifetime management

### Type System
- **No null pointers** - use `Option<T>` instead
- **Algebraic data types** - Enums can hold data
- **Pattern matching** - Exhaustive matching with `match`

### Error Handling
- **No exceptions** - use `Result<T, E>` type
- **Explicit error handling** - can't ignore errors easily

### Compilation
- **Ownership checking at compile time** - many runtime errors become compile errors
- **Zero-cost abstractions** - high-level features compile to efficient code

This cheatsheet covers the essential Rust syntax and concepts. The ownership system is the biggest mental shift from C++, but it provides memory safety without garbage collection!