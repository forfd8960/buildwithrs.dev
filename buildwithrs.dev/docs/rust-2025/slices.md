---
sidebar_position: 12
---

# Rust Slices

A slice in Rust is a dynamically-sized view into a contiguous sequence of elements, represented as `[T]`. Slices are one of Rust's most useful and fundamental data types, providing a way to reference a section of an array, vector, or string without taking ownership.

## Core Concepts

Slices are:

1. **References to data**: They don't own the data they point to
2. **Dynamically sized**: Their size is only known at runtime
3. **Contiguous**: Elements are laid out sequentially in memory
4. **Immutable by default**: Unless explicitly created as mutable with `&mut`

## Slice Syntax

```rust
// Basic slice syntax
let slice = &collection[start_index..end_index];

// Inclusive range (includes end_index)
let slice = &collection[start_index..=end_index];

```

The slice type is written as:

- `&[T]` for immutable slices
- `&mut [T]` for mutable slices
- `&str` for string slices (a special case)

## Creating Slices

### Array Slices

```rust
fn main() {
    // An array of integers
    let numbers = [1, 2, 3, 4, 5];

    // Create a slice of the 2nd and 3rd elements
    let slice = &numbers[1..3];

    println!("Array: {:?}", numbers);
    println!("Slice: {:?}", slice);  // Output: [2, 3]
}

```

### Slice Shorthand Notations

You can omit indices for more concise syntax [1](https://www.programiz.com/rust/slice):

```rust
fn main() {
    let numbers = [1, 2, 3, 4, 5];

    // Omit start index (starts from 0)
    let slice1 = &numbers[..3];  // Equivalent to &numbers[0..3]

    // Omit end index (goes to the end)
    let slice2 = &numbers[2..];  // Equivalent to &numbers[2..5]

    // Omit both (slice of the entire collection)
    let slice3 = &numbers[..];   // Equivalent to &numbers[0..5]

    println!("Slice1: {:?}", slice1);  // [1, 2, 3]
    println!("Slice2: {:?}", slice2);  // [3, 4, 5]
    println!("Slice3: {:?}", slice3);  // [1, 2, 3, 4, 5]
}

```

### String Slices

String slices (`&str`) are a special case of slices that work with UTF-8 encoded text:

```rust
fn main() {
    let message = String::from("Hello, world!");

    // Create a string slice
    let hello = &message[0..5];

    println!("Original string: {}", message);
    println!("Slice: {}", hello);  // Output: Hello
}

```

It's important to note that string slices must occur at valid UTF-8 character boundaries [1](https://doc.rust-lang.org/book/ch04-03-slices.html).

## Mutable Slices

You can create mutable slices to modify the underlying data:

```rust
fn main() {
    // Mutable array
    let mut colors = ["red", "green", "yellow", "white"];

    // Mutable slice
    let sliced_colors = &mut colors[1..3];

    println!("Original slice: {:?}", sliced_colors);  // ["green", "yellow"]

    // Modify the slice (and the original array)
    sliced_colors[1] = "purple";

    println!("Modified slice: {:?}", sliced_colors);  // ["green", "purple"]
    println!("Modified array: {:?}", colors);        // ["red", "green", "purple", "white"]
}

```

## Use Cases for &[T]

### 1. Function Parameters

Using slices as function parameters allows your functions to accept different types of collections:

```rust
// This function can take a slice from an array, Vec, or any contiguous sequence
fn sum(numbers: &[i32]) -> i32 {
    let mut total = 0;
    for num in numbers {
        total += num;
    }
    total
}

fn main() {
    let array = [1, 2, 3, 4, 5];
    let vector = vec![6, 7, 8, 9, 10];

    // Works with slices from both array and vector
    println!("Sum of array: {}", sum(&array[..]));       // 15
    println!("Sum of vector: {}", sum(&vector[..]));     // 40
    println!("Sum of partial array: {}", sum(&array[1..4])); // 9
}

```

This is a powerful pattern that makes your functions more flexible [5](https://www.reddit.com/r/rust/comments/xjjdkv/using_slices_instead_of_references_to_avoid/).

### 2. Pattern Matching and Slices

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let sentence = String::from("Hello world");
    let first = first_word(&sentence);
    println!("First word: {}", first);  // Output: Hello
}

```

### 3. Working with Partial Data

```rust
fn main() {
    let data = [10, 20, 30, 40, 50, 60];

    // Process data in chunks
    let chunk1 = &data[0..3];
    let chunk2 = &data[3..];

    process_chunk(chunk1);
    process_chunk(chunk2);
}

fn process_chunk(chunk: &[i32]) {
    println!("Processing chunk: {:?}", chunk);
    // Perform operations on the chunk...
}

```

### 4. Creating Views Without Copying

```rust
fn main() {
    let big_array = [0; 10000]; // An array with 10,000 zeros

    // Create multiple views into the same data without copying
    let first_half = &big_array[..5000];
    let second_half = &big_array[5000..];

    println!("First half length: {}", first_half.len());
    println!("Second half length: {}", second_half.len());
}

```

### 5. Slice Methods

Slices have many useful methods [2](https://doc.rust-lang.org/std/primitive.slice.html):

```rust
fn main() {
    let mut numbers = [1, 7, 3, 9, 5, 4, 8, 2, 6];

    // Sort a slice
    let slice = &mut numbers[..];
    slice.sort();

    println!("Sorted numbers: {:?}", numbers);  // [1, 2, 3, 4, 5, 6, 7, 8, 9]

    // Finding elements
    let slice = &numbers[..];
    println!("Contains 5: {}", slice.contains(&5));  // true

    // Split at a specific index
    let (left, right) = slice.split_at(4);
    println!("Left: {:?}, Right: {:?}", left, right);  // Left: [1, 2, 3, 4], Right: [5, 6, 7, 8, 9]

    // Iterating over chunks
    for chunk in slice.chunks(3) {
        println!("Chunk: {:?}", chunk);
    }
    // Output:
    // Chunk: [1, 2, 3]
    // Chunk: [4, 5, 6]
    // Chunk: [7, 8, 9]
}

```

## Key Points to Remember

1. Slices don't own data; they borrow it
2. String slices (`&str`) are a special case for text
3. Slices store both a pointer to the data and a length
4. Using slices for function parameters makes code more flexible
5. You cannot have a standalone slice without a reference (`&`) [4](https://stackoverflow.com/questions/76593802/can-rust-slice-type-t-be-used-without-reference-or-pointer)
6. Slices help prevent bugs by keeping track of both pointer and length information

Slices are fundamental to writing safe, efficient Rust code, enabling you to work with parts of collections without unnecessary copying or ownership transfers.