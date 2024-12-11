# process-fun-rs

A Rust library for easily running functions in separate processes with minimal boilerplate on Nix systems.

## Overview

`process-fun-rs` provides a simple macro-based approach to execute Rust functions in separate processes. It handles all the complexity of process spawning, argument serialization, and result communication, allowing you to focus on your business logic.

⚠️ **Important Notes:**
- This library currently only supports Nix-based systems
- When passing mutable data to a process function, modifications made within the process will not be reflected in the original data structure in the parent process. 

## Features

- Simple `#[process]` attribute macro for marking functions to create an additional version that runs in separate processes
- Automatic serialization/deserialization of function arguments and return values
- Type-safe process communication
- Error handling with custom error types
- Debug mode for troubleshooting process execution

## Usage

Add this to your `Cargo.toml`:

```toml
[dependencies]
process-fun = "0.1.0"
```

Basic example:

```rust
use process_fun::process;
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}

#[process]
pub fn add_points(p1: Point, p2: Point) -> Point {
    Point {
        x: p1.x + p2.x,
        y: p1.y + p2.y,
    }
}

fn main() {
    // Initialize the process-fun runtime
    process_fun::init_process_fun!();
    
    let p1 = Point { x: 1, y: 2 };
    let p2 = Point { x: 3, y: 4 };
    
    // This will execute in a separate process
    // Note: Any modifications to p1 or p2 inside the process
    // won't affect the original values in this process
    let result = add_points_process(p1, p2).unwrap();
    assert_eq!(result.x, 4);
    assert_eq!(result.y, 6);
}
```

Example demonstrating mutable data behavior:

```rust
use process_fun::process;
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Counter {
    value: i32,
}

#[process]
pub fn increment_counter(mut counter: Counter) -> Counter {
    counter.value += 1;
    counter
}

fn main() {
    let counter = Counter { value: 0 };
    
    // The counter is passed by value to the process
    let result = increment_counter_process(counter).unwrap();
    
    // Original counter is unchanged
    println!("Original counter value: {}", counter.value);  // Prints: 0
    println!("Result counter value: {}", result.value);     // Prints: 1
}
```

## How It Works

1. The `#[process]` attribute macro generates a wrapper function with `_process` suffix
2. When called, the wrapper function:
   - Serializes the arguments to JSON
   - Spawns a new process with the current executable
   - Passes a unique function hash and serialized arguments
   - Deserializes and returns the result

## Crate Structure

- `process-fun`: Main crate providing the public API
- `process-fun-core`: Core functionality and types
- `process-fun-macro`: Implementation of the `#[process]` attribute macro

## Requirements

- Nix-based operating system
- Functions marked with `#[process]` must:
  - Be public (`pub`)
  - Have arguments and return types that implement `Serialize` and `Deserialize`
  - Not take `self` parameters

## License

This project is licensed under the Apache 2 License - see the [LICENSE](LICENSE) file for details.