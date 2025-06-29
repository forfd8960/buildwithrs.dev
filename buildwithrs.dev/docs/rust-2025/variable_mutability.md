---
sidebar_position: 3
---

# Rust Variables and Mutability

In Rust, variables are immutable by default, which means that once a value is bound to a name, you can't change that value. This is one of Rust's core design principles that helps provide safety and enables easier concurrency.

## Basic Variable Declaration

To declare a variable in Rust, use the `let` keyword:

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    
    // The following would cause a compiler error:
    // x = 6; // error: cannot assign twice to immutable variable
}
```

## Making Variables Mutable

To make a variable mutable, add the `mut` keyword:

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    
    // Now we can change the value
    x = 6;
    println!("The value of x is now: {}", x);
}
```

## When to Use Mutable vs. Immutable Variables

### Use Immutable Variables (Default):

1. **When the value won't change** - This is the most straightforward case. If you know a value doesn't need to change, keep it immutable [1](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html).

2. **For better safety** - Immutability prevents accidental changes to variables, making your code more predictable and easier to reason about.

3. **For easier concurrency** - Immutable data can be safely shared between threads without locks.

4. **To communicate intent** - Using immutable variables by default signals to readers of your code that the value isn't meant to change.

### Use Mutable Variables:

1. **When values need to change over time** - Such as counters, accumulators, or state variables [1](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html).

2. **When implementing algorithms that require in-place modifications** - Some algorithms are more efficient when they can modify data structures in place.

3. **When using certain types that require internal mutability** - Some Rust types like random number generators require mutability for their methods to work, even if the variable itself isn't being reassigned [4](https://stackoverflow.com/questions/66722614/why-do-i-need-to-use-mut-when-the-variable-is-never-reassigned).

```rust
use rand::prelude::*;

fn main() {
    // Must be mutable because the random generator changes its internal state
    let mut rng = rand::thread_rng();
    let random_number = rng.gen_range(1..100);
    println!("Random number: {}", random_number);
}
```

## Constants vs. Immutable Variables

Rust also has constants, which are different from immutable variables:

```rust
fn main() {
    // Constant declaration
    const PI: f32 = 3.14159;
    println!("Value of PI = {}", PI);
}
```

Key differences:

- Constants use the `const` keyword and require type annotation
- Constants are always immutable (no `mut` option)
- Constants can only be set to constant expressions that can be computed at compile time
- Constants can be declared in any scope, including the global scope [1](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html)

## Variable Shadowing

Rust allows you to declare a new variable with the same name as a previous one, which is called "shadowing":

```rust
fn main() {
    let x = 5;
    let x = x + 1; // x is now 6
    
    {
        let x = x * 2; // x is now 12 in this scope
        println!("The value of x in the inner scope is: {}", x);
    }
    
    println!("The value of x is: {}", x); // x is still 6 here
}
```

Shadowing differs from mutability because:

1. You're creating a new variable, not changing an existing one
2. You can change the type of the value but reuse the same name
3. Each shadowing creates a new variable, so you're limited to the scope [1](https://doc.rust-lang.org/book/ch03-01-variables-and-mutability.html)

## Best Practices

1. **Default to immutable** - Start with immutable variables and only use `mut` when needed.
2. **Use clear naming** - Variable names should start with letters or underscores, can contain letters, digits, and underscores [3](https://www.programiz.com/rust/variables-mutability).
3. **Use constants for values that won't change** - Especially for values used throughout your program.
4. **Consider shadowing instead of mutation** - When you need to transform a value but don't need further mutations.

By following these principles, you'll write safer, more maintainable Rust code that better conveys your intentions.
