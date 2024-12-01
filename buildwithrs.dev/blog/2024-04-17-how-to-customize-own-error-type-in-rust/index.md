---
slug: how-to-custom-error-type-in-rust
title: How to Customize your Own Error Type in Rust
authors: [forfd8960]
tags: [rust, error]
---

Suppose you are writing a lexical analyzer(`Lexer`), and you want return some customized errors while the lexer scan tokens,
and there are 3 possibel error will happens:

- Not Supported Tokens.
- Invalid Strings.
- Invalid Nums.

SO you can define the errors with a rust `enum`:

```rust
pub enum LexerError {
    InvalidToken(char),
    InvalidString(String),
    InvalidNum(String),
}
```

<!-- truncate -->

And also you want the scan function can result it in the Result:

suppose the scan function looks like this:

```rust
pub fn scan_tokens(&mut self) -> Result<Vec<Token>, LexerError> {
    ...
}
```

In order to return the LexerError in the `Result<T, E>`, it need to implement the `Error` trait

```rust
/// Errors must describe themselves through the [`Display`] and [`Debug`]
/// traits. Error messages are typically concise lowercase sentences without
/// trailing punctuation:

...

pub trait Error: Debug + Display {
```

SO our current `LexerError` type need to implement `Debug` and `Display` trait.

For `Debug` trait we just derive it in the enum, and for `Disply` trait we can implement it as:

- Derive Debug trait for your New Error Type

```rust
#[derive(Debug)]
pub enum LexerError {
    InvalidToken(char),
    InvalidString(String),
    InvalidNum(String),
}
```

- Implement `std::error::Erro` and `std::fmt::Display` for your new Error Type.

```rust
impl std::error::Error for LexerError {}

impl std::fmt::Display for LexerError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        match self {
            LexerError::InvalidToken(msg) => write!(f, "{}", msg),
            LexerError::InvalidString(msg) => write!(f, "{}", msg),
            LexerError::InvalidNum(msg) => write!(f, "{}", msg),
        }
    }
}
```

## Summary

to customize your own **Error Type**, need to do followings:

- Derive Debug trait for new error: `#[derive(Debug)]`.

- Implement `std::error::Error` and `std::fmt::Display` trait for new error.
