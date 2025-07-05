---
sidebar_position: 15
---

# Rust Error Handling with Result

Rust approaches error handling differently than many other languages by using return values rather than exceptions. This makes error handling explicit and forces developers to consider failure cases [6](https://web.mit.edu/rust-lang_v1.25/arch/amd64_ubuntu1404/share/doc/rust/html/book/first-edition/error-handling.html).

## Understanding `Result<T, E>`

The `Result<T, E>` enum is Rust's primary mechanism for handling recoverable errors [2](https://doc.rust-lang.org/book/ch09-00-error-handling.html). It has two variants:

```rust
enum Result<T, E> {
    Ok(T),  // Success case containing a value of type T
    Err(E), // Error case containing an error of type E
}
```

When a function might fail, it returns a `Result` type instead of throwing an exception [1](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html).

## Basic Result Handling with match

Here's a simple example of opening a file which could fail:

```rust
use std::fs::File;

fn main() {
    let file_result = File::open("hello.txt");
    
    let file = match file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
    
    // Use the file here if successful
}
```

## Handling Different Error Types

You can match on specific error types to handle them differently:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let file_result = File::open("hello.txt");
    
    let file = match file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

In this example, if the file doesn't exist, we try to create it. For any other error, we panic [1](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html).

## Shortcuts for Result Handling

### Using unwrap_or_else

The previous example can be rewritten more concisely using closures:

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

### Using unwrap and expect

For quick prototyping or when you're certain an operation will succeed, you can use these shortcuts:

```rust
// unwrap: Returns the value or panics if there's an error
let file = File::open("hello.txt").unwrap();

// expect: Like unwrap, but with a custom error message
let file = File::open("hello.txt").expect("Failed to open hello.txt");
```

These methods should be used sparingly in production code as they cause panics on errors [3](https://doc.rust-lang.org/rust-by-example/error.html).

## The ? Operator

The `?` operator is a concise way to propagate errors. When applied to a `Result`, it returns the `Ok` value if successful or returns the `Err` value from the current function [5](https://www.reddit.com/r/rust/comments/1cm5bes/error_handling_the_right_way/).

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_file_contents() -> Result<String, io::Error> {
    let mut file = File::open("hello.txt")?;
    let mut contents = String::new();
    file.read_to_string(&mut contents)?;
    Ok(contents)
}

fn main() {
    match read_file_contents() {
        Ok(contents) => println!("File contents: {}", contents),
        Err(error) => println!("Error reading file: {:?}", error),
    }
}
```

This is much cleaner than using nested `match` expressions. The `?` operator automatically converts the error type if necessary (using the `From` trait) [4](https://bitfieldconsulting.com/posts/rust-errors-option-result).

## Chaining Multiple Operations

The `?` operator really shines when you need to perform multiple fallible operations:

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();
    File::open("hello.txt")?.read_to_string(&mut username)?;
    Ok(username)
}
```

## Combining Result with Option

You can convert between `Result` and `Option` using methods like `ok()` and `ok_or()`:

```rust
fn find_user(id: u64) -> Option<User> {
    // Implementation that returns Some(user) or None
}

fn get_user_by_id(id: u64) -> Result<User, String> {
    find_user(id).ok_or(format!("No user found with id: {}", id))
}
```

## Custom Error Types

For larger applications, it's common to define custom error types:

```rust
use std::fmt;
use std::error::Error;
use std::io;
use std::num::ParseIntError;

#[derive(Debug)]
enum AppError {
    IoError(io::Error),
    ParseError(ParseIntError),
    Custom(String),
}

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::IoError(e) => write!(f, "IO error: {}", e),
            AppError::ParseError(e) => write!(f, "Parse error: {}", e),
            AppError::Custom(s) => write!(f, "Error: {}", s),
        }
    }
}

impl Error for AppError {}

impl From<io::Error> for AppError {
    fn from(error: io::Error) -> Self {
        AppError::IoError(error)
    }
}

impl From<ParseIntError> for AppError {
    fn from(error: ParseIntError) -> Self {
        AppError::ParseError(error)
    }
}

fn read_and_parse() -> Result<i32, AppError> {
    let mut content = String::new();
    File::open("number.txt")?.read_to_string(&mut content)?;
    let number: i32 = content.trim().parse()?;
    Ok(number)
}
```

With the `From` implementations, the `?` operator will automatically convert specific errors to your custom error type.

## Best Practices

1. Use `Result` for recoverable errors and `panic!` for unrecoverable errors
2. Consider using the `?` operator to propagate errors
3. Use `unwrap()` and `expect()` only in prototyping or when failure is impossible
4. Create custom error types for complex applications
5. Provide meaningful error messages

By following these principles, you'll write more robust Rust code that handles errors gracefully and explicitly.
