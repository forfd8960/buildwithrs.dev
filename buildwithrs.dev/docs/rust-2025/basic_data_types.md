---
sidebar_position: 4
---

# Rust Basic Data Types

In Rust, every value has a specific data type which dictates how it's stored in memory and what operations can be performed on it. Rust is a statically typed language, meaning that the compiler needs to know the types of all variables at compile time [1](https://doc.rust-lang.org/book/ch03-02-data-types.html).

## Data Type Categories

Rust data types are divided into two main categories:

1. **Scalar Types**: Single value types
2. **Compound Types**: Types that group multiple values together

Let's focus on scalar types and their usage in Rust.

## Scalar Types

Scalar types represent single values. Rust has four primary scalar types [1](https://doc.rust-lang.org/book/ch03-02-data-types.html)[3](https://dev.to/francescoxx/rust-data-types-1mlg):

### 1. Integer Types

Integers are numbers without a fractional component. Rust provides a variety of integer types with different sizes and sign properties:

| Length | Signed | Unsigned |
|--------|--------|----------|
| 8-bit  | i8     | u8       |
| 16-bit | i16    | u16      |
| 32-bit | i32    | u32      |
| 64-bit | i64    | u64      |
| 128-bit| i128   | u128     |
| arch   | isize  | usize    |

**Key points about integers:**

- **Signed vs. Unsigned**: Signed integers can represent both positive and negative values, while unsigned integers can only represent non-negative values [1](https://doc.rust-lang.org/book/ch03-02-data-types.html).
- **Default type**: The default integer type is `i32` [1](https://doc.rust-lang.org/book/ch03-02-data-types.html).
- **Range**: Each type has a specific range. For example:
  - `i8`: -128 to 127
  - `u8`: 0 to 255 [3](https://dev.to/francescoxx/rust-data-types-1mlg)

**Integer literals in Rust can be written in various formats:**

```rust
fn main() {
    let decimal = 98_222;      // Decimal
    let hex = 0xff;            // Hexadecimal
    let octal = 0o77;          // Octal
    let binary = 0b1111_0000;  // Binary
    let byte = b'A';           // Byte (u8 only)
    
    println!("Decimal: {}", decimal);
    println!("Hexadecimal: {}", hex);
    println!("Octal: {}", octal);
    println!("Binary: {}", binary);
    println!("Byte: {}", byte);
}
```

```bash
> cargo run --package rust-2025 --bin bst
warning: `rust-2025` (bin "bst") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.99s
     Running `target/debug/bst`
Decimal: 98222
Hexadecimal: 255
Octal: 63
Binary: 240
Byte: 65
```

### 2. Floating-Point Types

Floating-point types are numbers with decimal points. Rust has two floating-point types:

- `f32`: 32-bit floating-point (single precision)
- `f64`: 64-bit floating-point (double precision)

**Key points about floating-point types:**

- The default type is `f64` since it's roughly the same speed as `f32` on modern CPUs but provides more precision [1](https://doc.rust-lang.org/book/ch03-02-data-types.html)[3](https://dev.to/francescoxx/rust-data-types-1mlg).

```rust
fn main() {
    let x = 2.0;        // f64 by default
    let y: f32 = 3.0;   // f32 with explicit type annotation
    
    // Basic arithmetic operations
    let sum = x + (y as f64);
    let difference = x - (y as f64);
    let product = x * (y as f64);
    let quotient = x / (y as f64);
    
    println!("Sum: {}", sum);
    println!("Difference: {}", difference);
    println!("Product: {}", product);
    println!("Quotient: {}", quotient);
}
```

```bash
> cargo run --package rust-2025 --bin bst

warning: `rust-2025` (bin "bst") generated 1 warning
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/bst`
Sum: 5
Difference: -1
Product: 6
Quotient: 0.6666666666666666
```

### 3. Boolean Type

The Boolean type in Rust is represented by `bool`. It can have only two values: `true` or `false`.

```rust
fn main() {
    let t = true;
    let f: bool = false;  // With explicit type annotation
    
    // Using booleans in conditional statements
    if t {
        println!("This will print");
    }
    
    if !f {
        println!("This will also print");
    }
}
```

```bash
> cargo run --package rust-2025 --bin bst

This will print
This will also print
```

### 4. Character Type

The `char` type in Rust represents a Unicode Scalar Value, which means it can encode much more than just ASCII:

- It's 4 bytes in size
- Can represent letters, accented characters, digits, emoji, etc.
- Unicode range: U+0000 to U+D7FF and U+E000 to U+10FFFF [3](https://dev.to/francescoxx/rust-data-types-1mlg)

```rust
fn main() {
    let c = 'z';           // ASCII character
    let z = 'Z';           // Another ASCII character
    let heart = '❤';       // Unicode character
    let emoji = '😻';      // Emoji (also Unicode)
    
    println!("Characters: {}, {}, {}, {}", c, z, heart, emoji);
    
    // Iterating over characters in a string
    for character in "Hello, 世界!".chars() {
        println!("{}", character);
    }
}
```

```bash
> cargo run --package rust-2025 --bin bst

Characters: z, Z, ❤, 😻
H
e
l
l
o
,
 
世
界
```

## Type Inference and Annotations

Rust can often infer the type of a variable based on its value and usage, but sometimes you need to add type annotations:

```rust
// Type inference - Rust figures out the type
let number = 42;  // i32 by default

// Type annotation - explicitly specifying the type
let number: u32 = 42;

// When parsing strings, type annotation is usually required
let parsed_number: u32 = "42".parse().expect("Not a number!");
```

If you don't specify a type when it's needed, Rust will show a helpful error message asking for type annotations [1](https://doc.rust-lang.org/book/ch03-02-data-types.html).

## Integer Overflow Handling

Rust handles integer overflow differently depending on compilation mode:

- In debug mode: Rust includes overflow checks that cause your program to panic at runtime if overflow occurs
- In release mode: Rust performs two's complement wrapping (e.g., for a u8, 256 becomes 0, 257 becomes 1) [1](https://doc.rust-lang.org/book/ch03-02-data-types.html)

For explicit overflow handling, Rust provides several methods:

- `wrapping_*` methods: Wrap in all modes
- `checked_*` methods: Return `None` if overflow occurs
- `overflowing_*` methods: Return the value and a boolean indicating overflow
- `saturating_*` methods: Saturate at the minimum or maximum values [1](https://doc.rust-lang.org/book/ch03-02-data-types.html)

This comprehensive overview covers the scalar types in Rust and their usage. Understanding these basic types is essential for writing effective and safe Rust code.
