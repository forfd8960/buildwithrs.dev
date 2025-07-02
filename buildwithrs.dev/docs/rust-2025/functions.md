---
sidebar_position: 6
---

# Rust Functions and Parameters

Functions are fundamental building blocks in Rust that allow you to organize code into reusable units. Rust code uses snake case as the conventional style for function and variable names, where all letters are lowercase and underscores separate words.

## Basic Function Syntax

In Rust, functions are declared using the `fn` keyword:

```rust
fn function_name(parameter1: type1, parameter2: type2) -> return_type {
    // Function body
}
```

### Simple Function Example

```rust
fn main() {
    println!("Hello, world!");
    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

When you run this program, it will output:
```
Hello, world!
Another function.
```

The execution flows in the order statements appear in the `main` function [1](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html).

## Functions with Parameters

Parameters are special variables that are part of a function's signature. When you define a function with parameters, you can provide concrete values (arguments) when calling the function [1](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html).

### Basic Parameter Example

```rust
fn main() {
    greet("Alice");
    greet("Bob");
}

fn greet(name: &str) {
    println!("Hello, {}!", name);
}
```

Output:

```bash
Hello, Alice!
Hello, Bob!
```

### Multiple Parameters

You can define functions with multiple parameters by separating them with commas:

```rust
fn main() {
    print_info("Alice", 30);
    print_info("Bob", 25);
}

fn print_info(name: &str, age: u32) {
    println!("{} is {} years old", name, age);
}
```

Output:

```bash
Alice is 30 years old
Bob is 25 years old
```

### Type Annotations in Function Signatures

In Rust, you must declare the type of each parameter in function signatures. This is a deliberate design decision that helps the compiler provide better error messages and reduces the need for type annotations elsewhere in your code [1](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html).

## Functions with Return Values

Functions can return values back to the caller. In Rust, you specify the return type after an arrow (`->`):

```rust
fn main() {
    let sum = add(5, 6);
    println!("The sum is: {}", sum);
}

fn add(a: i32, b: i32) -> i32 {
    a + b  // Note: no semicolon here - this is an expression that returns a value
}
```

Output:

```bash
The sum is: 11
```

### Implicit Return with Expressions

In Rust, the last expression in a function will be returned implicitly if there's no semicolon. This is because Rust is an expression-based language [1](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html) [2](https://doc.rust-lang.org/rust-by-example/fn.html).

```rust
fn calculate_area(width: u32, height: u32) -> u32 {
    // The last expression is returned (no semicolon)
    width * height
}
```

### Explicit Return with the `return` Keyword

You can also use the `return` keyword to return from a function early:

```rust
fn max(a: i32, b: i32) -> i32 {
    if a > b {
        return a;
    }
    b  // Implicit return if a is not greater than b
}
```

## Use Cases for Rust Functions and Parameters

### 1. Code Organization and Reuse

Functions help organize code into logical, reusable units:

```rust
fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    
    println!("Numbers: {:?}", numbers);
    println!("Sum: {}", sum(&numbers));
    println!("Average: {:.2}", average(&numbers));
    println!("Minimum: {}", min(&numbers));
    println!("Maximum: {}", max(&numbers));
}

fn sum(numbers: &[i32]) -> i32 {
    numbers.iter().sum()
}

fn average(numbers: &[i32]) -> f64 {
    if numbers.is_empty() {
        return 0.0;
    }
    sum(numbers) as f64 / numbers.len() as f64
}

fn min(numbers: &[i32]) -> i32 {
    *numbers.iter().min().unwrap_or(&0)
}

fn max(numbers: &[i32]) -> i32 {
    *numbers.iter().max().unwrap_or(&0)
}
```

### 2. Function Composition

Building complex operations by combining simpler functions:

```rust
fn main() {
    let radius = 5.0;
    let area = calculate_circle_area(radius);
    println!("A circle with radius {} has area {:.2}", radius, area);
    
    let side_length = 4.0;
    let area = calculate_square_area(side_length);
    println!("A square with side length {} has area {:.2}", side_length, area);
}

fn calculate_circle_area(radius: f64) -> f64 {
    let pi = std::f64::consts::PI;
    pi * square(radius)
}

fn calculate_square_area(side: f64) -> f64 {
    square(side)
}

fn square(x: f64) -> f64 {
    x * x
}
```

### 3. Domain-Specific Logic Encapsulation

Grouping related functionality:

```rust
fn main() {
    let username = "alice_smith";
    let email = "alice@example.com";
    let password = "password123";
    
    if validate_username(username) && validate_email(email) && validate_password(password) {
        println!("User registration data is valid");
        create_user(username, email, password);
    } else {
        println!("Invalid user registration data");
    }
}

fn validate_username(username: &str) -> bool {
    let valid_length = username.len() >= 3 && username.len() <= 20;
    let valid_chars = username.chars().all(|c| c.is_alphanumeric() || c == '_');
    
    valid_length && valid_chars
}

fn validate_email(email: &str) -> bool {
    // Simple email validation
    email.contains('@') && email.contains('.')
}

fn validate_password(password: &str) -> bool {
    password.len() >= 8
}

fn create_user(username: &str, email: &str, password: &str) {
    println!("Creating user: {}", username);
    // Actual user creation logic would go here
    println!("User created successfully!");
}
```

### 4. Using Functions as Abstraction Barriers

Hiding implementation details behind a function interface:

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    
    let doubled = transform_numbers(&numbers, double);
    let squared = transform_numbers(&numbers, square);
    
    println!("Original: {:?}", numbers);
    println!("Doubled: {:?}", doubled);
    println!("Squared: {:?}", squared);
}

fn transform_numbers(numbers: &[i32], transform_fn: fn(i32) -> i32) -> Vec<i32> {
    numbers.iter().map(|&num| transform_fn(num)).collect()
}

fn double(x: i32) -> i32 {
    x * 2
}

fn square(x: i32) -> i32 {
    x * x
}
```

### 5. Functions with Default Values Using Option Parameters

Implementing default parameter values in Rust:

```rust
fn main() {
    greet_user("Alice", None);
    greet_user("Bob", Some("Admin"));
}

fn greet_user(name: &str, role: Option<&str>) {
    // Unwrap the role or use "User" as default
    let user_role = role.unwrap_or("User");
    println!("Hello, {}! Your role is: {}", name, user_role);
}
```

### 6. Functions with Mutable Parameters

Allowing functions to modify their parameters:

```rust
fn main() {
    let mut numbers = vec![1, 2, 3, 4, 5];
    println!("Before: {:?}", numbers);
    
    add_one(&mut numbers);
    println!("After adding one: {:?}", numbers);
    
    multiply_by_two(&mut numbers);
    println!("After multiplying by two: {:?}", numbers);
}

fn add_one(numbers: &mut Vec<i32>) {
    for num in numbers.iter_mut() {
        *num += 1;
    }
}

fn multiply_by_two(numbers: &mut Vec<i32>) {
    for num in numbers.iter_mut() {
        *num *= 2;
    }
}
```

### 7. Functions with Slice Parameters for Flexibility

Using slices to accept different collection types:

```rust
fn main() {
    // Working with an array
    let array = [1, 2, 3, 4, 5];
    println!("Sum of array: {}", sum(&array));
    
    // Working with a vector
    let vector = vec![10, 20, 30, 40, 50];
    println!("Sum of vector: {}", sum(&vector));
    
    // Working with a slice of a vector
    println!("Sum of vector slice: {}", sum(&vector[1..4]));
}

// This function accepts any slice of i32 values
fn sum(numbers: &[i32]) -> i32 {
    numbers.iter().sum()
}
```

### 8. Using Functions as Building Blocks for More Complex Algorithms

```rust
fn main() {
    let numbers = vec![9, 2, 7, 3, 6, 4, 5, 1, 8];
    
    println!("Original: {:?}", numbers);
    println!("Even numbers: {:?}", filter_numbers(&numbers, is_even));
    println!("Odd numbers: {:?}", filter_numbers(&numbers, is_odd));
    println!("Prime numbers: {:?}", filter_numbers(&numbers, is_prime));
}

fn filter_numbers(numbers: &[i32], predicate: fn(i32) -> bool) -> Vec<i32> {
    numbers.iter().filter(|&&n| predicate(n)).cloned().collect()
}

fn is_even(n: i32) -> bool {
    n % 2 == 0
}

fn is_odd(n: i32) -> bool {
    n % 2 != 0
}

fn is_prime(n: i32) -> bool {
    if n <= 1 {
        return false;
    }
    if n <= 3 {
        return true;
    }
    if n % 2 == 0 || n % 3 == 0 {
        return false;
    }
    
    let mut i = 5;
    while i * i <= n {
        if n % i == 0 || n % (i + 2) == 0 {
            return false;
        }
        i += 6;
    }
    true
}
```

### 9. Early Returns for Error Handling

Using early returns to handle error conditions:

```rust
fn main() {
    let result = divide(10, 2);
    match result {
        Ok(value) => println!("Result: {}", value),
        Err(msg) => println!("Error: {}", msg),
    }
    
    let result = divide(10, 0);
    match result {
        Ok(value) => println!("Result: {}", value),
        Err(msg) => println!("Error: {}", msg),
    }
}

fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        return Err("Cannot divide by zero".to_string());
    }
    
    Ok(a / b)
}
```

### 10. Functions that Don't Return a Value (Unit Type)

In Rust, functions that don't explicitly return a value actually return the unit type, written as `()` [2](https://doc.rust-lang.org/rust-by-example/fn.html):

```rust
fn main() {
    let result = print_and_return_nothing();
    println!("Function returned: {:?}", result);
}

// This function returns the unit type ()
fn print_and_return_nothing() -> () {
    println!("I don't return anything meaningful");
    // Implicitly returns () here
}
```

## Advanced Function Concepts

### Generic Functions

Functions that work with any type that satisfies certain constraints:

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];
    println!("First: {:?}", first(&numbers));
    
    let chars = vec!['a', 'b', 'c'];
    println!("First: {:?}", first(&chars));
}

// Generic function that works with any type
fn first<T: Copy>(slice: &[T]) -> Option<T> {
    if slice.is_empty() {
        None
    } else {
        Some(slice[0])
    }
}
```

### Function Pointers

Using functions as first-class values:

```rust
fn main() {
    // Array of function pointers
    let operations: [fn(i32, i32) -> i32; 4] = [add, subtract, multiply, divide_safe];
    
    let a = 10;
    let b = 5;
    
    for (i, operation) in operations.iter().enumerate() {
        let operation_name = match i {
            0 => "Addition",
            1 => "Subtraction",
            2 => "Multiplication",
            3 => "Division",
            _ => "Unknown",
        };
        
        println!("{}: {} and {} = {}", operation_name, a, b, operation(a, b));
    }
}

fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn subtract(a: i32, b: i32) -> i32 {
    a - b
}

fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

fn divide_safe(a: i32, b: i32) -> i32 {
    if b == 0 { 0 } else { a / b }
}
```

Functions are a core component in Rust programming, providing a way to organize code, promote reuse, and create clear interfaces between different parts of your program.