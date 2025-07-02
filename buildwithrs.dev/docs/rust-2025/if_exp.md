---
sidebar_position: 7
---

# Control Flow: `if` Expression

In Rust, `if` expressions are a fundamental part of control flow, allowing you to execute different blocks of code based on conditions. Here's a breakdown of how to use `if`, `else`, and `else if` in Rust, along with examples:

**1. Basic `if` Expression**

An `if` expression executes a block of code only if a condition is true [3](https://www.programiz.com/rust/if-else).

```rust
fn main() {
    let number = 10;

    if number > 0 {
        println!("{} is greater than 0", number);
    }
}
```

**2. `if...else` Expression**

The `if...else` expression allows you to execute one block of code if the condition is true and another block of code if the condition is false [3](https://www.programiz.com/rust/if-else).

```rust
fn main() {
    let number = -2;

    if number > 0 {
        println!("{} is greater than 0", number);
    } else {
        println!("{} is less than or equal to 0", number);
    }
}
```

**3. `if...else if...else` Expression**

You can evaluate multiple conditions using `else if` expressions. This is useful when you need to handle more than two possible outcomes [3](https://www.programiz.com/rust/if-else).

```rust
fn main() {
    let number = -2;

    if number > 0 {
        println!("{} is greater than 0", number);
    } else if number == 0 {
        println!("{} is equal to 0", number);
    } else {
        println!("{} is less than 0", number);
    }
}
```

**4. `if` as an Expression**

In Rust, `if` blocks are also expressions, meaning they can return values.  All branches must return the same type [1](https://doc.rust-lang.org/rust-by-example/flow_control/if_else.html).

```rust
fn main() {
    let n = 5;

    let big_n = if n < 10 && n > -10 {
        println!("number is a small number, increase ten-fold");
        10 * n
    } else {
        println!("number is a big number, halve the number");
        n / 2
    };

    println!("{} -> {}", n, big_n);
}
```

**5. Matching on Options**

`if let` is useful for checking if an `Option` contains a value.

```rust
fn main() {
    let optional_value: Option<i32> = Some(42);

    if let Some(value) = optional_value {
        println!("The value is: {}", value);
    } else {
        println!("The optional value is None");
    }
}
```

**6. Using `if` with `match`**

You can combine `if` statements with `match` for more complex control flow [4](https://www.reddit.com/r/learnrust/comments/y9qgg1/what_is_the_proper_way_to_deal_with_variables/).

```rust
fn main() {
    let value = 3;

    match value {
        1 => println!("Value is 1"),
        2 => println!("Value is 2"),
        _ if value > 2 => println!("Value is greater than 2"),
        _ => println!("Value is something else"),
    }
}
```

These examples cover the main ways to use `if`, `else`, and `else if` in Rust. They demonstrate how to create conditional logic, return values from `if` expressions, and work with `Option` types.
