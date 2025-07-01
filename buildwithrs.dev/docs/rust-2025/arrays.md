---
sidebar_position: 6
---

# Rust Arrays

Arrays are one of Rust's compound data types that allow you to store multiple values of the same type in a single data structure. Unlike tuples, which can store values of different types, arrays require all elements to be of the same type [3](https://doc.rust-lang.org/rust-by-example/primitives.html).

## Array Basics

An array in Rust is defined by its element type and length, both of which are fixed at compile time. The type signature for an array is `[T; N]` where `T` is the element type and `N` is the length (a constant expression).

### Creating Arrays

```rust
fn main() {
    // Array with explicit type annotation
    let numbers: [i32; 5] = [1, 2, 3, 4, 5];
    
    // Array with type inference
    let colors = ["red", "green", "blue", "yellow", "purple"];
    
    // Creating an array with repeated values
    let zeros = [0; 10]; // Creates [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    
    println!("First number: {}", numbers[0]);
    println!("Second color: {}", colors[1]);
    println!("Tenth zero: {}", zeros[9]);
}
```

### Accessing Array Elements

Array elements are accessed using indexing (zero-based):

```rust
fn main() {
    let weekdays = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"];
    
    // Accessing elements
    println!("First day: {}", weekdays[0]);
    println!("Middle day: {}", weekdays[2]);
    println!("Last day: {}", weekdays[4]);
    
    // Rust performs bounds checking at runtime
    // The following would panic if uncommented:
    // println!("Invalid index: {}", weekdays[5]);
}
```

### Arrays and Slices

You can create slices (references to a section of an array):

```rust
fn main() {
    let numbers = [1, 2, 3, 4, 5];
    
    // Creating a slice of the entire array
    let all_numbers = &numbers[..];
    
    // Creating a slice of part of the array
    let middle_numbers = &numbers[1..4]; // [2, 3, 4]
    
    println!("All numbers: {:?}", all_numbers);
    println!("Middle numbers: {:?}", middle_numbers);
}
```

## Use Cases for Arrays in Rust

### 1. Fixed-Size Collections

When you need a collection of data with a known, fixed size that won't change:

```rust
fn main() {
    // Storing days of the week
    let days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
    
    // Months of the year
    let months = [
        "January", "February", "March", "April", "May", "June", 
        "July", "August", "September", "October", "November", "December"
    ];
    
    println!("Third day: {}", days[2]);
    println!("Sixth month: {}", months[5]);
}
```

### 2. Performance-Critical Code

Arrays allow for stack allocation and have no overhead compared to heap-allocated collections like `Vec`:

```rust
fn calculate_moving_average(values: &[f64; 5]) -> f64 {
    let sum: f64 = values.iter().sum();
    sum / values.len() as f64
}

fn main() {
    let temperatures = [22.5, 23.1, 23.8, 22.9, 23.5];
    let average = calculate_moving_average(&temperatures);
    
    println!("Moving average temperature: {:.2}°C", average);
}
```

### 3. Buffer Management

Arrays are useful for creating fixed-size buffers:

```rust
use std::io;

fn read_data() -> io::Result<()> {
    // Create a buffer to read into
    let mut buffer = [0u8; 1024];
    
    // Read up to 1024 bytes from standard input
    let bytes_read = io::stdin().read(&mut buffer)?;
    
    println!("Read {} bytes: {:?}", bytes_read, &buffer[..bytes_read]);
    Ok(())
}
```

### 4. Matrix and Grid-Based Data

For 2D data like a game board or an image:

```rust
fn main() {
    // Simple 3x3 game board (like tic-tac-toe)
    let mut board = [
        [' ', ' ', ' '],
        [' ', ' ', ' '],
        [' ', ' ', ' ']
    ];
    
    // Make some moves
    board[0][0] = 'X';
    board[1][1] = 'O';
    board[0][1] = 'X';
    
    // Print the board
    for row in &board {
        println!("{:?}", row);
    }
}
```

### 5. Lookup Tables

Pre-computed values for fast lookup:

```rust
fn main() {
    // Pre-computed squares for quick lookup (0² to 9²)
    let squares = [0, 1, 4, 9, 16, 25, 36, 49, 64, 81];
    
    // Using the lookup table
    for i in 0..10 {
        println!("{}² = {}", i, squares[i]);
    }
    
    // Pre-computed trigonometric values (in radians)
    let sin_table = [0.0, 0.5, 0.707, 0.866, 1.0]; // sin(0°), sin(30°), sin(45°), sin(60°), sin(90°)
    let cos_table = [1.0, 0.866, 0.707, 0.5, 0.0]; // cos(0°), cos(30°), cos(45°), cos(60°), cos(90°)
    
    println!("sin(60°) ≈ {}", sin_table[3]);
    println!("cos(60°) ≈ {}", cos_table[3]);
}
```

### 6. Function Arguments for Fixed-Size Datasets

When you want to ensure a function receives exactly the expected amount of data:

```rust
// Function that processes exactly 5 sensor readings
fn process_sensor_data(readings: [f32; 5]) -> f32 {
    // Discard highest and lowest, average the rest
    let mut sorted = readings;
    sorted.sort_by(|a, b| a.partial_cmp(b).unwrap());
    
    // Take middle 3 values and average them
    (sorted[1] + sorted[2] + sorted[3]) / 3.0
}

fn main() {
    let sensor_readings = [12.5, 14.3, 15.0, 12.8, 13.2];
    let filtered_value = process_sensor_data(sensor_readings);
    
    println!("Filtered sensor value: {:.2}", filtered_value);
}
```

### 7. Bit Manipulation and Flags

Using arrays to represent bit flags:

```rust
fn main() {
    // Bit flags represented as an array of booleans
    // [keyboard, mouse, display, network, audio, battery]
    let mut device_status = [true, true, true, false, true, true];
    
    // Device failure detected
    device_status[3] = true; // Network is now working
    device_status[5] = false; // Battery has failed
    
    // Check device status
    for (i, &status) in device_status.iter().enumerate() {
        let device_name = match i {
            0 => "Keyboard",
            1 => "Mouse",
            2 => "Display",
            3 => "Network",
            4 => "Audio",
            5 => "Battery",
            _ => "Unknown"
        };
        
        println!("{}: {}", device_name, if status { "OK" } else { "Failed" });
    }
}
```

### 8. Pattern Matching with Arrays

Arrays work well with Rust's pattern matching:

```rust
fn interpret_input(key_sequence: [char; 3]) -> &'static str {
    match key_sequence {
        ['q', 'u', 'i'] => "Quit command recognized",
        ['n', 'e', 'w'] => "New document command",
        ['s', 'a', 'v'] => "Save command recognized",
        ['c', 'u', 't'] => "Cut command recognized",
        _ => "Unknown command sequence"
    }
}

fn main() {
    println!("{}", interpret_input(['q', 'u', 'i']));
    println!("{}", interpret_input(['h', 'e', 'l']));
}
```

### 9. Working with Fixed-Size Binary Data

For protocols or file formats with fixed-size structures:

```rust
fn main() {
    // Example: RGB color value (3 bytes)
    let rgb_red = [255u8, 0, 0];
    let rgb_green = [0u8, 255, 0];
    let rgb_blue = [0u8, 0, 255];
    
    // Example: IPv4 address (4 bytes)
    let local_host = [127u8, 0, 0, 1];
    let broadcast = [255u8, 255, 255, 255];
    
    // Example: MAC address (6 bytes)
    let mac_address = [0x00u8, 0x1A, 0x2B, 0x3C, 0x4D, 0x5E];
    
    println!("Red RGB: {:?}", rgb_red);
    println!("IPv4 localhost: {}.{}.{}.{}", 
             local_host[0], local_host[1], local_host[2], local_host[3]);
    println!("MAC address: {:02X}:{:02X}:{:02X}:{:02X}:{:02X}:{:02X}",
             mac_address[0], mac_address[1], mac_address[2],
             mac_address[3], mac_address[4], mac_address[5]);
}
```

### 10. Constant-Time Access

When you need O(1) access to elements:

```rust
fn main() {
    // Calendar mapping day number to day name
    let days = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"];
    
    fn get_day_name(day_number: usize) -> Option<&'static str> {
        if day_number >= 1 && day_number <= 7 {
            Some(days[day_number - 1])
        } else {
            None
        }
    }
    
    // Using the function
    match get_day_name(3) {
        Some(day) => println!("Day 3 is {}", day),
        None => println!("Invalid day number")
    }
}
```

## Limitations of Arrays

1. **Fixed size**: Arrays cannot be resized once created.
2. **Same type**: All elements must be of the same type.
3. **Stack allocation**: Large arrays can cause stack overflow (use `Vec` for large collections).

## When to Use Arrays vs. Vectors

- Use arrays when:
  - The size is fixed and known at compile time
  - You need the performance benefits of stack allocation
  - You want to ensure the collection never changes size
  
- Use vectors (`Vec<T>`) when:
  - The size may change at runtime
  - You need to store a large amount of data (heap allocation)
  - You need operations like push, pop, etc.

Arrays in Rust provide a low-level, efficient way to work with collections of fixed size, making them ideal for performance-critical code and situations where the size is known in advance.
